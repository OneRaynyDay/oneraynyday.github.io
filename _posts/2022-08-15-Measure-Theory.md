---
published: true
title: Probability Tidbits 2 - Measure Theory
use_math: true
category: math
layout: default
---

# Probability Tidbits 2 - Measure Theory
Given a set $X$, $\mathcal{A}$ is a **$\sigma$-algebra** on the set if:

$$
\begin{align}
\emptyset, X \in \mathcal{A} \\
A \in \mathcal{A} \implies A^c \in \mathcal{A} \\
A_i \in \mathcal{A} \implies \bigcup_{i} A_i \in \mathcal{A}
\end{align}
$$

where the union could be potentially countable. The combination of the set and its $\sigma$-algebra $(X, \mathcal{A})$ is called a **measurable space**. Trivial examples of $\mathcal{A}$ include $\{\emptyset, X\}$ and $P(X)$ i.e. the power set. There exists a mapping $\sigma: P(X) \to P(X)$ such that for any subset $U \subset X$, $\sigma(U)$ produces the smallest $\sigma$-algebra such that $U \subset \sigma(U)$. So in other words, we can construct a $\sigma$-algebra given an arbitrary subset of $X$.

Also, given a set $X$, $\tau$ is a **topology** on the set if:

$$
\begin{align}
\emptyset, X \in \mathcal{A} \\
A_i \in \tau \implies \bigcup_{i} A_i \in \tau \\
A_i \in \tau \implies \bigcap_{i}^N A_i \in \tau, N \in \mathbb{N}
\end{align}
$$

where the union could be countable but the intersection of sets is finite. The combination of the set and its topology $(X, \tau)$ is called a **topological space**. 

Why did I mention these two? In particular, we're interested in a particular $\sigma$-algebra called $B(X) = \sigma(\tau)$ or the Borel $\sigma$-algebra on a topology $\tau$ of X, which as noted before is defined as the smallest $\sigma$-algebra containing $\tau$. We need a measurable space because we're going to be dealing with probability measures, and we need a topological space because we need some concept of metric in $X$ in order to quantify how close elements in $X$ are with each other (so we can build some analytic intuition on continuity which is important for stochastic calculus).

Given the measurable space $(X, \mathcal{A})$, a map $\mu: B(X) \to [0, \infty]$ is a **measure** if:

$$
\begin{align}
\mu(\emptyset) = 0 \\
\forall i \neq j, A_i \bigcap A_j = \emptyset \implies \mu(\bigcup_i A_i) = \sum_i \mu(A_i)

\end{align}
$$

Intuitively $\mu$ looks really close to $P$ which maps an outcome to a probability, and indeed we're almost there. Using a change in notation here, **probability space $(\Omega, \mathcal{F}, P)$** is a triple where $\Omega$ can be thought of as $X$, $\mathcal{F}$ is a $\sigma$-algebra on $\Omega$, and $P$ is our **probability measure**. In addition to the definition of measure above, we need $P(\Omega) = 1$ for it to be a probability measure. When it comes to continuous distributions, we often consider the outcome space $\Omega$ to be the continuum, i.e. $\mathbb{R}$, $\mathcal{F}$ its corresponding Borel $\sigma$-algebra on the typical topology on $\mathbb{R}$. The probability measure is dependent on the probability distribution. For example, the univariate normal distribution measure is defined as $P([a,b]) = \Phi(b) - \Phi(a)$ for any $a < b$, $a,b \in \mathbb{R}$.

A **measurable function** $f: X \to Y$ is defined as a function that maps one measurable space $(X, \mathcal{A})$ to another measurable space $(Y, \mathcal{B})$ and preserves structure. Specifically, $\forall E \in \mathcal{B}, f^{-1}(E) \in \mathcal{A}$. An example of a measurable function is $f:\mathbb{R} \to \mathbb{R}, f(x) = x^2 \quad \forall x \in \mathbb{R}$, since for example $f^{-1}((0, 9)) = (-3, 3)$. These intervals are measurable in the standard $(\mathbb{R}, B(\mathbb{R}))$ measurable space. Also, a measurable function is not the same thing as a measure - measurable functions map between two measurable spaces, while a measure maps measurable sets of a measurable space into the positive extended real number line!

In this sense **random variables** are measurable functions mapping from $\Omega$, the set in a probability space $(\Omega, \mathcal{F}, P)$, to $\mathbb{R}$ (typically). The standard notation of random variable equaling a real value $x$ can thus be thought of as the preimage of the random variable on $x$:

$$
P(X = x, x\in \mathbb{R}) = P(\{\omega : X(\omega) = x\}) = P(X^{-1}(x))
$$

Supposed the preimage was $n$ elements of the set $\Omega$:

$$
X^{-1}(x) = \{\omega_1, \omega_2, ..., \omega_n\}
$$

Then, someone could also just say:

$$
P(\{\omega_1, \omega_2, ..., \omega_n\}) = P(X^{-1}(x))
$$

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
