---
published: true
title: Probability Tidbits 6 - Kolmolgorovs 0-1 Law
use_math: true
category: math
layout: default
---

# Probability Tidbits 6 - Kolmogorov's 0-1 Law
In the [4th tidbit](https://oneraynyday.github.io/math/2022/09/09/Surely-and-Often/) we discussed the definition of **almost surely** and **infinitely often**. In the discussion of the converse of the second **Borel-Cantelli lemma**, we referenced the **Kolmogorov 0-1 Law**, but didn't prove it or even state the general result. **The law states that for independent random variables $$\{X_n\}_n$$ its tail $\sigma$-algebra is $P$-trivial.** Let's define some terms in that word salad.

## Independence
The definition of independence taught in undergrad probability theory concerns events, in that for event $A, B \in \mathcal{F}$ where $\mathcal{F}$ is some $\sigma$-algebra, these events are independent if:

$$
P(A \cap B) = P(A)P(B)
$$

Typically, for events $A_1,...,A_N$ to be independent, they must be jointly independent (pairwise independence isn't enough):

$$
P(\bigcap_nA_n) = \prod_nP(A_n)
$$

In the context of measure theoretic probability theory, we often concern ourselves with independence on the sub $\sigma$-algebra level. Suppose we have two sub $\sigma$-algebras $\mathcal{H}, \mathcal{G} \subset \mathcal{F}$, then they are independent if:

$$
P(H \cap G) = P(H)P(G) \quad \forall H \in \mathcal{H}, G \in \mathcal{G}
$$

When we say two random variables are independent, we mean their corresponding generated $\sigma$-algebras are independent. As we discussed in the [3rd tidbit](https://oneraynyday.github.io/math/2022/08/23/Pi-Systems/) working with $\sigma$-algebras is difficult and proving the above independence is not easy. Thankfully $\pi$-systems, a much simpler construct often embedded in $\sigma$-algebras, can come to the rescue here. Recall that **two measures that agree on the same $\pi$-system will be the same measure on the $\sigma$-algebra generated by that $\pi$-system.** Suppose the two sub $\sigma$-algebras $\mathcal{H}, \mathcal{G} \subset \mathcal{F}$ have corresponding $\pi$-systems $\mathcal{I}, \mathcal{J}$ such that $\sigma(\mathcal{I}) = \mathcal{H}, \sigma(\mathcal{J}) = \mathcal{G}$. A natural question is: **if $\mathcal{I},\mathcal{J}$ are independent, does that make $\mathcal{H}, \mathcal{G}$ independent as well?** The answer is yes - in fact this is an iff statement. We know that

$$
P(I \cap J) = P(I)P(J)
$$

If we were to fix $I$, then we can see that the measure $\mu'(J) := P(I \cap J)$ agrees with $\mu$ fixed on $\mathcal{J}$. These two measures are the same, so it'll be the same measure on $\sigma(\mathcal{J}) = \mathcal{G}$, which implies independence on the $\sigma$-algebra level. So now we know that $\mathcal{I}$, a $\pi$-system, is independent with $\mathcal{G}$, a $\sigma$-algebra. Now let's fix some $G \in \mathcal{G}$ and apply the same argument to $I$, so that $\mathcal{G}$ is independent with $\sigma(\mathcal{I}) = \mathcal{H}$. Now we have $\mathcal{H}, \mathcal{G}$ independent.

## Tail $\sigma$-algebra
Recall in the [5th tidbit](https://oneraynyday.github.io/math/2022/09/10/Random-Variables/) we said that a random variable $X$ can generate a $\sigma$-algebra:

$$
\sigma(X) := \sigma(\{\{\omega \in \Omega: X(\omega) \in B\} : B \in \mathcal{B}\})
$$

Suppose we have a sequence of random variables $$\{X_n\}_n$$ then collections of these random variables can generate a sigma algebra:

$$
\sigma(X_1, .., X_N) = \sigma(\{\{\omega \in \Omega: X_i(\omega) \in B\} : B \in \mathcal{B}, i \leq N\})
$$
Let's define a specific type of generated $\sigma$-algebra created by countable collections:

$$
\mathcal{T}_n = \sigma(X_n,X_{n+1},...), \mathcal{T} = \bigcap_n \mathcal{T}_n
$$
Here, $\mathcal{T}$ is the **tail $\sigma$-algebra** consisting of **tail events**.

## $P$-trivial

This term, though not as commonly used, is pretty simple. The most trivial probability measure is one where the probability can be either 1 or 0. In the measurable space $(\mathbb{R}, \mathcal{B})$, a trivial measure $P$ is one where $P(X = c) = 1$ for some $c \in \mathbb{R}$, and consequently 0 everywhere else. Plotting its corresponding distribution function, this looks like a right-continuous step function (recall from [tidbit 5](https://oneraynyday.github.io/math/2022/09/10/Random-Variables/), all distribution functions are right continuous).

## Kolmogorov's 0-1 Law
Now that we have all the setup, let's figure out why Kolmogorov's law is true. To remind us: **The law states that for independent random variables $$\{X_n\}_n$$ its tail $\sigma$-algebra $\mathcal{T}$ is $P$-trivial.** Let's start off with these $\sigma$-algebras:

$$
\mathcal{X}_N = \sigma(X_1,...,X_N), \mathcal{T}_N = \sigma(X_{N+1}, X_{N+2}, ...)
$$

These are basically "the measurable events before and after $N$". Intuitively, $\mathcal{X}_N \perp \mathcal{T}_N$, but it's kind of hard to reason about these huge $\sigma$-algebras, so let's make a $\pi$-system for each:

$$
\mathcal{I}_N = \{\omega: X_i(\omega) \leq x_i \forall i \leq N\}, \mathcal{J}_N = \{\omega: X_i(\omega) \leq x_i \forall i > N\}
$$

These are $\pi$-systems since:

$$
\begin{align}
A = \{\omega: X_i(\omega) \leq a_i \forall i \leq N\} \in \mathcal{I}_N \\
B = \{\omega: X_i(\omega) \leq b_i \forall i \leq N\} \in \mathcal{I}_N \\
A \cap B = \{\omega: X_i(\omega) \leq \text{min}(a_i, b_i) \forall i \leq N\} \in \mathcal{I}_N
\end{align}
$$

and obviously $$\mathcal{I}_N \neq \emptyset$$. A similar argument can be made for $$\mathcal{J}_N$$. These $\pi$-systems are independent since each $X_i$ are independent and by our previous result in the independence section, we have $$\mathcal{X}_N \perp \mathcal{T}_N$$. But we know that $$\mathcal{T} \subset \mathcal{T}_N \forall N$$! This means $$\mathcal{T} \perp \mathcal{X}_N \forall N$$. Now let's push $$N \to \infty$$ - is $$\mathcal{X}_\infty \perp \mathcal{T}$$? We know that $$
\mathcal{X}_N \subset \mathcal{X}_{N+1} \forall N
$$, and as with most "monotonically increasing sets of sets" this is probably a $\pi$-system!

$$
\begin{align}
A, B \in \bigcup_n \mathcal{X}_n \\
\implies A \in \mathcal{X}_a, B \in \mathcal{X}_b \text{ for some } a, b \in \mathbb{N} \\
\implies A \cap B \in \mathcal{X}_{min(a,b)} \subset \bigcup_n \mathcal{X}_n
\end{align}
$$

Let's denote this $\pi$-system as $$\mathcal{K} := \bigcup_n \mathcal{X}_n$$. We know that $$\mathcal{K} \perp \mathcal{T}$$, since for any element in $\mathcal{K}$, we have that it belongs to some $$\mathcal{X}_n$$ which is independent of $\mathcal{T}$. We also know that $$\sigma(\mathcal{K}) \perp \mathcal{T}$$ since $\pi$-systems independence extend to its generated $\sigma$-algebra. But wait - $$\mathcal{T} \subset \sigma(\mathcal{K})$$! This is true since $$\mathcal{T} := \bigcap_n \sigma(X_n, X_{n+1}, ...) \subset \sigma(X_1,X_2,...)$$. This means $$\mathcal{T} \perp \mathcal{T}$$, so that

$$
T \in \mathcal{T}, P(T \cap T) = P(T)P(T) \implies P(T) \in \{0, 1\}
$$

Thus $\mathcal{T}$ is $P$-trivial. With Kolmogorov's law, we now know that if a tail event $T$ will either happen with probability 0 or 1 - there is no in between (kind of like a law of excluded middle). We used this in the converse of the second Borel-Cantelli lemma to show that if the probability of the tail event $\text{limsup}_{n \to \infty} E_n$ **not** occurring is greater than 0, then the tail event must have probability 0.

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
