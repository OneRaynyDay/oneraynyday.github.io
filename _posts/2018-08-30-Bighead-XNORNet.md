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

# Introduction

Over the summer, I interned at Airbnb's machine learning infrastructure team, working on Airbnb's to-be-opensourced library called **`Bighead`**. Think Uber's `Michaelangelo`, in that it's an end-to-end machine learning framework, meant to wrap around most of your typical machine learning workflow, from data ingestion, to training, to hyperparameter selection, visualization, and finally deployment. When it becomes opensource, you can figure out the details for yourself.

An argument against end-to-end machine learning frameworks is that you would need to work exclusively in their environment, and for other frameworks that are not completely compliant with its API, we would need to add wrappers (like `tensorflow`, `SpaCy`, etc). 

Howeever, a similar argument for end-to-end machine learning frameworks is that because you have a homogenous wrapper interface, the user _shouldn't really care about which framework underneath they use*, just that it works, and works well._

If we have our own ecosystem, we can build our own data ingestion pipeline, optimize them however we want, and do cool things, like **speed up machine learning inference by swapping out frameworks depending on which stage of the pipeline you are current executing.**

_* - Unless they're hardcore fanboys of specific frameworks. In that case, I got nothing._ 

# Integer 8-bit quantization

Nvidia has a library for forward-inference called `TensorRT`. It's neat because it uses underlying GPU intrinsics for optimization (`INT8 GEMM DP4A`, etc), and so on Nvidia specific GPU's, it runs _very fast_. Nvidia has done some work in quantizing the weights and inputs of the neural network, down from `float32` to `float16` or `int8`. For `float32` to `float16`, you are essentially performing `static_cast<float16>(float_32_number)`. For `int8`, it's a little different:
$$
\gamma^* = argmin_\gamma D_{KL}(Q||P_\gamma) \\
I^W_{ij} = cast_{int8}(\frac{W_{ij}}{\gamma^*}) \\
W \approx I^W * \gamma^*
$$
where $Q$ is some unknown distribution(explained later), and $P$ is our distribution. $\gamma$ is a parameter in $P$, and we optimize the kullback-leibler convergence between these two distributions with respect to $\gamma$. 

Now, let's explain Q and P.

For layers of a neural network, we usually generate these things called _activation values_, from functions called _activation functions_. The most common one is the sigmoid:
$$
sigmoid(x) = \frac{1}{1+e^{-x}}
$$
With some data distribution $D = {x_i}_{i=0}^N$, we have some kind of activation function $f$ that does $f(D) = Q$. This is a generic function f, with some unknown distribution $D$. 

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