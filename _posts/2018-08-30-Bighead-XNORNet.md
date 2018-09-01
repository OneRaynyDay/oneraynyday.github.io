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

where $Q$ is some unknown distribution(explained later), and $P$ our distribution. $\gamma$ is a parameter in $P$, and we optimize the kullback-leibler convergence between these two distributions with respect to $\gamma$. The $cast_{int8}$ function is a thresholding function:

$$
cast_{int8}(x) = max(min(-127, 2( \lfloor \frac{(2^8-1)(x+1)}{2(2^8-1)} \rfloor - \frac{1}{2})), 127)
$$

Okay, maybe it's better expressed with code:

```python
def cast_int8(x):
    return max(min(-127, int(x)), 127)
```

Now, let's explain $Q$ and $P$.

For layers of a neural network, we usually generate these things called _activation values_, from functions called _activation functions_. The most common one is the sigmoid:

$$
\sigma (x) = \frac{1}{1+e^{-x}}
$$

... but technically almost anything could be considered an activation function. With some data distribution $D = \{x_i\}_{i=0}^N$, we have some kind of activation function $f$ where $f(D) = P$. This is a generic function $f$, with some unknown distribution $D$. 

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

The performance of this model in a single GPU usually yields ~1-2x faster inference speed. On a `p3.8xlarge` (with 4 beefy Volta GPU's), we can get to at most 3x faster on `Resnet50`. However, can we discretize this even further?

# Bitwise quantization

Bitwise quantization is similar to `int8` quantization. The approximation is as follows:

$$
A, B \in \Re^{NxM} \\
\mathcal{B}^X \in \{-1, +1\}^{NxM}
\alpha, \beta \in \Re
$$

The approximation is as follows:

$$
AB \approx \mathcal{B}^{A} \mathcal{B}^{B}\alpha\beta
$$

One can find the best binary array and magnitudes by solving the following least squares equation:

$$
argmin_{\mathcal{B}^{A},\alpha} ||A - \mathcal{B}^{A}*\alpha||^2
$$

After some calculations, the answer is (intuitively):

$$
\mathcal{B}^{X}_{ij} = signum(X)_{ij} \\
\alpha = \frac{||A||_1}{N}
$$

Multiplying two of these approximations is simple, and also leads to the optimal approximation of the two original matrices multiplied together:

$$
A*B \approx \mathcal{B}^{A}*\mathcal{B}^{B} * \alpha * \beta
$$

And if the above were numbers:

$$
a*b \approx sign(a)*sign(b) * \alpha * \beta
$$

---

The reason I added the number example is because matrix multiplication can be expressed as a series of dot products. If we can optimize the dot product kernel, it means we can optimize the matrix kernel as well.

If we had a number in `float32`:

```
a = [sign of a | mantissa of a | exponent of a]
a * b = [sign a XNOR sign b | mantissa a * mantissa b | exp a + exp b]
```

If we had a number in ${-1, +1}$, and mapped $-1$ to $0$, and $1$ to $1$ in bits:

```
a = [sign of a]
b = [sign of b]
a * b = a XNOR b
```

This means all of our multiplications can be done via `XNOR`'s! This means we can pack our data into bits and do a vectorized xor on multiple elements at once. This should definitely be faster than performing floating point multiplication in theory. In addition, a dot product is just a combination of multiplication and sum reduction. If we use `XNOR` for multiplication, we need an equivalent for sum reduction. Fortunately, there is a native intrinsic `popcount` instruction that allows one to count the number of `1`'s in bytes(32 bytes in AVX2, 64 bytes in AVX512, etc).

A naive approach to the dot product would be:

```python
x = ~ (a ^ b) # xnor
result = popcount(x) # popcount
```

which is going to be the equivalent of our dot product.

# Approaches to Implement XNORNet

`BLAS` is a very ancient, and established linear algebra framework. It stands for :

- `B`asic 
- `L`inear 
- `A`lgebra 
- `S`ubprograms

, and many libraries implement their subroutines based off of `BLAS`'s interface. Some of the fastest variants of `BLAS` are `MKL` from Intel, `ATLAS`, and `OpenBLAS`. The most common subprograms deep learning uses are:

- `dot` (dot product between 2 vectors)
- `gemv`/`gevm`, `ge`neral `m`atrix `v`ector
- `gemm`, `ge`neral `m`atrix `m`atrix

Because many deep learning frameworks are glued to `BLAS` libraries, we are faced with two paths:

1. Implement a BLAS routine for bit-wise matrix multiplication, and convince some 3rd party deep learning library to incorporate our changes.
2. Make a forward-inference framework ourselves, and have the flexibility to replace `gemm` with `xnorgemm` anywhere we want.

## Forward-inference Framework

### Linear Algebra Backend

We wanted our library to interface directly with `NumPy`, which is the standard general numerical computing stack in Python along with `SciPy`. However, directly using `NumPy` will incur huge copy costs and reduced optimization, as well as the curse of GIL locking in Python.

We decided that we needed something performant and expressive, that had good bindings with `NumPy`, and so one of the promising candidates was `xtensor`, from the QuantStack team. It also had a variety of optimizations that we were looking for:

- Support for `NumPy` bindings
- Support for N-dimensional tensors (something that Armadillo and Eigen could not provide out-of-the-box)
- Support for lazy operators, thus reducing temporary copies and support efficient type checking using templates during compile time
- Support for black-box `simd` intrinsic operations
- Use modern C++11 for move semantics and compile-time generated fixed-size expressions using type traits built into the standard library.

So now we have decided on our linear algebra library and language of choice, how can we dynamically express the neural network in Python, but get it to work in C++?

### Python to C++ Transpiler

As a primary goal of usability, we wanted the end-user to not spend too much time learning about our framework, but rather use their familiar Python Keras/MXNet/Tensorflow/Pytorch high-level layers, which are all similar in API. Just like how MXNet compiles its python graph into NNVM, and Tensorflow compiles its graph into LLVM, we compile our python graph into C++. Transpiling something that looks like:

TODO: Add example here














