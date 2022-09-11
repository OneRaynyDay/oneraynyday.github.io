---
published: true
title: Probability Tidbits 4 - Almost Surely and Infinitely Often
use_math: true
category: math
layout: default
---

# Probability Tidbits 4 - Almost Surely and Infinitely Often

Suppose we have a probability space $(\Omega, \mathcal{F}, P)$, an **event** is some $E \in \mathcal{F}$ and **outcomes** are $\omega \in \Omega$. It'd be nice to characterize behavior events on uncountably infinite measurable spaces like $(\mathbb{R}, B(\mathbb{R}))$.

Let's roll a six-sided die. The set is $$\Omega = \{1,2,3,4,5,6\}$$. An example outcome is rolling the number 3. The outcome of rolling 3 could be in event of $$E = \{\text{rolling an odd number}\}$$.

An event $E \in \mathcal{F}$ happens **almost surely** if $P(E) = 1$. Suppose you were to throw a dart on a board - denote the event of you hitting the center as $E$. $P(E) = 0$ obviously, so $P(E^c) = 1$. Here, $E$ happens **almost never** and $E^c$ happens **almost surely**.

Suppose we have a sequence $E_n$ of events. We want to denote that $E_n$ happens **infinitely often**:


$$
\text{limsup}_{n \to \infty} E_n = \bigcap_n\bigcup_{k \geq n} E_k
$$


The inner countable union means "fix an $n$, take all the outcomes that could happen in the rest of the sequence". The outer intersection pushes $n$ to infinity, thus filtering out events that could only happen finitely many times. An example $E_n$ is $E_n := \{\text{the n-th coin flip is heads}\}$. In a fair coin toss, this event is intuitively likely to happen half of the time and as we take $n$ to infinity this should happen infinitely often. The **Borel-Cantelli lemmas** state:


$$
\begin{align}
\sum_n P(E_n) < \infty \implies P(\text{limsup}_{n \to \infty} E_n) = 0 \\
\sum_n P(E_n) = \infty, (E_n)_n \text{ independent} \implies P(\text{limsup}_{n \to \infty} E_n) = 1
\end{align}
$$


The proof for the first statement is pretty simple and we'll omit the second:

$$
P(\bigcup_nE_n) \leq \sum_n P(E_n) < \infty
$$

The monotonic sequence $S_k = \sum_i^k P(E_i)$ converging to some $S$ implies that $\forall \epsilon \exists N \in \mathbb{N}$ such that $S - S_n < \epsilon \forall n > N$. Here, $S - S_n = \sum_{m\geq n}P(E_m) \geq P(\bigcap_{m\geq n} \bigcup_{k \geq m} E_k)$, so the probability of some outcome happening infinitely often converges to zero.

Note that if the second statement's summation isn't unbounded:

$$
\sum_n P(E_n) = S < \infty
$$

We have:

$$
\begin{align}
& log(1-P(E_n)) \leq (1-P(E_n)) - 1 \quad \text{(taylor expansion, concavity)} \\
& \implies log(1-P(E_n)) \geq P(E_n) \\
& \implies \sum_n log(1-P(E_n)) \geq \sum_n P(E_n) = S \\
& \implies log(\prod_n 1-P(E_n)) \geq S > 0 \\
& \implies \prod_n (1-P(E_n)) > 0 \\
& \implies P(\bigcup_n \bigcap_{k \geq n} E_k^c) > 0
\end{align}
$$

as in $$P(\text{limsup}_{n \to \infty} E_n) \neq 1$$ or the events **doesn't happen infinitely often**, then by Kolmolgorov's 0-1 law (covered in the [6th tidbit](https://oneraynyday.github.io/math/2022/09/11/Kolmogorovs-0-1-Law/)) the events will happen finitely often and thus $$P(\text{limsup}_{n \to \infty} E_n) = 0$$.

## Infinity, Monkeys, and Shakespeare
We have countably many monkeys at your service, each with their own type writer minding their own business. Say we made them all start typing at $T = 0$ and waited. Now that we're equipped with the second Borel-Cantelli lemma, we see that the event $A_i = \{\text{i-th monkey types shakespeare}\}$ is extremely unlikely, but $P(A_i) > 0$. We also know that these monkeys mind their own business so the events $A_i \perp A_j$ , so therefore:

$$
\sum_i P(A_i) = \infty \implies P(\text{limsup}_{n \to \infty} A_i) = 1
$$

In other words, *almost surely*, the monkeys would produce an entire Shakespeare collection.

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
