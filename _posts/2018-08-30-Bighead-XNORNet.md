---
published: true
title: Airbnb Bighead - Speeding Up Inference
use_math: true
category: ML
layout: default
---

# Table of Contents

* TOC
{:toc}
This article is WIP.

_The opinions expressed in this post are my own and not necesarily those of my employer (i.e. don't fire me pls)_

# Introduction

Over the summer, I interned at Airbnb's machine learning infrastructure team, working on Airbnb's to-be-opensourced library called **`Bighead`**. Think Uber's `Michaelangelo`, in that it's an end-to-end machine learning framework, meant to wrap around most of your typical machine learning workflow, from data ingestion, to training, to hyperparameter selection, visualization, and finally deployment. When it becomes opensource, you can figure out the details for yourself.

An argument against end-to-end machine learning frameworks is that you would need to work exclusively in their environment, and for other frameworks that are not completely compliant with its API, we would need to add wrappers (like `tensorflow`, `SpaCy`, etc). 

However, a similar argument for end-to-end machine learning frameworks is that because you have a homogenous wrapper interface, the user _shouldn't really care about which framework underneath they use, just that it works well, is easy to deploy, and easy to visualize and interpret._ 

If we have our own ecosystem, we can build our own data ingestion pipeline, optimize them however we want, and do cool things, like **speed up machine learning inference by swapping out frameworks depending on which stage of the pipeline you are current executing.**

The scoped context of this blog doesn't do `Bighead` enough justice(there are many other features that make it a great library), so check it out when it's available!

# Integer 8-bit quantization

Nvidia has a library for forward-inference called `TensorRT`. It's neat because it uses underlying GPU intrinsics for optimization (`INT8 GEMM DP4A`, etc), and so on Nvidia specific GPU's, it runs _very fast_. Nvidia has done some work in quantizing the weights and inputs of the neural network, down from `float32` to `float16` or `int8`. For `float32` to `float16`, you are essentially performing `static_cast<float16>(float_32_number)`. For `int8`, it's a little different:

$$
\gamma^* = argmin_\gamma D_{KL}(P||Q_\gamma) \\
I^W_{ij} = cast_{int8}(\frac{W_{ij}}{\gamma^*}) \\
W \approx I^W * \gamma^*
$$

where Q is some unknown distribution(explained later), and P is our distribution. \gamma is a parameter in P, and we optimize the kullback-leibler convergence between these two distributions with respect to \gamma. 

Now, let's explain Q and P.

For layers of a neural network, we usually generate these things called _activation values_, from functions called _activation functions_. The most common one is the sigmoid:

$$
sigmoid(x) = \frac{1}{1+e^{-x}}
$$

With some data distribution $D = \{x_i\}_{i=0}^N$, we have some kind of activation function $f$ where $f(D) = P$. This is a generic function $f$, with some unknown distribution $D$. 

Now, if we discretize all elements involved in $f$, including the input and the weights required for any operator, we get back a function $f_{int8}$, and $f_{int8}(D) * \gamma = Q_\gamma$. We minimize the KL divergence between these two distributions, $Q_\gamma$ and $P$.

TL;DR: We find the best approximation of $I^W$ with respect to $\gamma$ and the current activation function $f$.

## Details

We refer to `MXNet`'s source code, in which I made a PR in, to explain exactly how the quantization step happens. The link to the file is [here](https://github.com/apache/incubator-mxnet/blob/master/python/mxnet/contrib/quantization.py), and the PR is [here](https://github.com/apache/incubator-mxnet/pull/11833).

```python
hist, hist_edges = np.histogram(arr, bins=num_bins, range=(-th, th))
zero_bin_idx = num_bins // 2
num_half_quantized_bins = num_quantized_bins // 2
assert np.allclose(hist_edges[zero_bin_idx] + 
                   hist_edges[zero_bin_idx + 1],
                   0, rtol=1e-5, atol=1e-7)
```

Here, we create a histogram to approximate our activation function. The KL divergence between two continuous-valued distributions is usually performed in software via approximate binning. Each entry of the histogram is a bin entry. i.e. If you have 2 bins, then:

$$
D_{KL}([1,1]||[1,1]) = 0
$$

Because these 2 distributions are the same.

We are about to enter the important `for` loop that decides what threshold, i.e. $\gamma$ is optimal. We run through all reasonable $\gamma$'s and pick the one that gives the lowest KL divergence. The reasonable $\gamma$'s are between $[0, max(\|x_i\|) \forall i]$. Any elements outside of the distribution will be absorbed to the two corners of the distribution.

An example:

```
# Sampled numbers, sorted without loss of generality
[-10, -2, 1, 8, 12]
# If gamma = 2, and 4 bins, then we have histograms of
[|x <= -1|, |-1 < x <= 0|, |0 < x <= 1|, |1 < x|]
= [2, 0, 1, 2]
```

So the below is the process:

```python
# generate reference distribution p
p = sliced_nd_hist.copy()
assert p.size % 2 == 1
assert p.size >= num_quantized_bins
# put left outlier count in p[0]
left_outlier_count = np.sum(hist[0:p_bin_idx_start])
p[0] += left_outlier_count
# put right outlier count in p[-1]
right_outlier_count = np.sum(hist[p_bin_idx_stop:])
p[-1] += right_outlier_count
# is_nonzeros[k] indicates whether hist[k] is nonzero
is_nonzeros = (p != 0).astype(np.int32)
```

Here, we discretize our reference distribution $P$, which is retrieved from $f(D)$. In this case, we have generated `arr` which is $P$. Recall that in practice, we discretize our distributions into bins so we can do a discrete KL divergence calculation.

```python
# calculate how many bins should be merged to generate quantized distribution q
num_merged_bins = sliced_nd_hist.size // num_quantized_bins
# merge hist into num_quantized_bins bins
for j in range(num_quantized_bins):
    start = j * num_merged_bins
    stop = start + num_merged_bins
    quantized_bins[j] = sliced_nd_hist[start:stop].sum()
    
quantized_bins[-1] += sliced_nd_hist[
    num_quantized_bins * num_merged_bins:].sum()
```

Here, we generate our distribution $Q$, by binning up the current scope of numbers within $[-\gamma, \gamma]$ into the # of bins you can fit in an `int8`, which is $\|[-127,-126,...,127]\| = 2^8-1 = 255$ bins.

Now that we have retrieved `q` and `p`, let's do a KL divergence on them. Using scikit:

```python
divergence[i - num_half_quantized_bins] = stats.entropy(p, q)
```

And so we have found our best threshold by choosing the minimum entropy from the above expression.

There are some extra details that I left out, about smoothing the distribution to prevent divide by 0 errors and etc. Read the code for more information.

_Note: The `contrib` folder for most frameworks is volatile, so the code above may not be an exact 1:1 at the time that you are reading it._

The performance of this model in a single GPU usually yields ~1-2x faster inference speed. On a `p3.8xlarge` (with 4 beefy Volta GPU's), we can get to at most 3x faster on `Resnet50`.

# Bitwise quantization

Bitwise quantization is similar to `int8` quantization. The approximation is as follows:

$$
A, B \Re^{NxM} \\
\mathcal{B}^X \in \{-1, +1\}^{NxM} \\
\alpha, \beta \in \Re \\
\mathcal{B}^{X}_{ij} = signum(X)_{ij} \\
\alpha = \frac{||A||_1}{N} \\
AB \approx \mathcal{B}^{A} \mathcal{B}^{B}\alpha\beta
$$

WIP