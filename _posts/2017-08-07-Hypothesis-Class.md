---
published: false
title: Statistical Learning Theory - Hypothesis Class
use_math: true
layout: default
---

# What is a hypothesis?

A hypothesis, can be **thought of as a function**. However, it's not necessary. A function maps something to something else, like $f: \Re^n \to \Re$. For every input, we know there exists a single output. In mathematical terms, this is called **injective** [TODO: CHECK]. However, in our world of unknowns, it's not known whether a point will map to more than one coordinate, depending on the **underlying noise**. 

Our definition of noise, is the unknown consequences of the outside environment that we failed to capture in our data. This noise makes the deterministic idea of a function fall apart, so instead of:

$$
f(x_n)
$$

We now take the stochastic analog:

$$
p(y_n|x_n)
$$

Which gives us a probability distribution of all outputs.

Now abstractions aside, a hypothesis in our case is just a mathematical model that tries to map x's to y's, like linear regression (usually $f : \Re^n \to \Re$), or a support vector machine (usually $f : \Re^n \to \{-1, +1\}$). 

But **be careful**, you can't say "linear models" is a hypothesis. It's too broad - we need to specify the specific weights, i.e. "linear model with coefficients $x0 = 1$, $x1 = 0.2$, etc.". Thus, we call "linear models" a hypothesis set.

# Recap

Hoeffding, as we talked about before, tries to analyze the bound of the difference between the empirical risk and the expected risk of a hypothesis, $f$.

What we managed to get was:

$$
P(|R_{N}(f) - R(f)| > \epsilon) \leq 2e^{-2N\epsilon^2}
$$

This gives us confidence about whether our model, trained on our training data, will be able to generalize, and express the underlying data distribution, $x, y \sim p_\theta$. 

This is great and all, but **this is only one hypothesis**. For a general class of hypotheses, like support vectors, how can we make a similar argument?

# Uniform Bound

The most basic way to plug Hoeffding into a general class of hypothesis, which we will denote $ \mathcal{H} $, is to assume that there exists no overlap between the bad hypotheses. A bad hypothesis is one where:

$$
|R_{N}(f_bad) - R(f)| \geq \epsilon
$$

for some $\delta$. 


