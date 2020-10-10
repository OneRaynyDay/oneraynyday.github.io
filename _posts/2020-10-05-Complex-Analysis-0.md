---
published: true
title: Complex Analysis - Metric spaces review
use_math: true
category: math
layout: default
---

This is a very informal set of notes that I'll be maintaining while taking Terrence Tao's complex analysis class.

# Fields, order, sequences

$\mathbb{R}, \mathbb{C}$ are fields, which are sets equipped with addition and multiplication. In addition, they exhibit associativity, commutativity, identity, inverses for both addition and multiplication, as well as distributivity of multiplication over addition. $\mathbb{R}$ is an **ordered field**, which obeys the [order axioms](https://en.wikipedia.org/wiki/Ordered_field) with $<$. Because $\mathbb{R}$ is an ordered field, we usually use $sup$ and $inf$ to analyze the boundedness of sequences and series. However, $\mathbb{C}$ is not an ordered field.

We can easily prove it using $i \in \mathbb{C}$. Due to [trichotomy](https://en.wikipedia.org/wiki/Trichotomy_(mathematics)), $i < 0$ or $i = 0$ or $i > 0$. $i = 0$ is trivially absurd. If $i > 0$ then the order axiom $a, b > 0 \implies ab > 0$ is violated (pick $i$ for both $a, b$). If $i < 0$ then we pick $-i > 0$ as both $a, b$ and we violate the same axiom again.

$\mathbb{C}$ has candidate functions to define distance, which makes it a metric space(explained in the next section). A sequence in $\mathbb{C}$, $\\{z_n\\}$ **converges** to $w \in \mathbb{C}$ if $lim_{n \to \infty} \|z_n - w\| = 0$.

Because $\mathbb{C}$ is **complete** (which we'll prove later), **cauchy** sequences which have the property:

$$
|z_n-z_m| \to 0 \ \text{as} \ n,m \to \infty
$$

Must also converge to some value $w \in \mathbb{C}$. Convergence and cauchy are equivalent in a complete metric space.

---

# Metric space properties

A metric space is a set $S$ equipped with a function $d: S \times S \to \mathbb{R}$. The function $d$ tells us the "distance" between two points in the set. It has the properties:

- $d(x, y) = 0 \iff x = y$ (positive definite)
- $d(x, y) = d(y, x)$ (symmetric)
- $d(x, z) \leq d(x, y) + d(y, z)$ (triangle inequality)

In this case, $\mathbb{C}$ is a metric space if we consider the **bilinear form** of $\mathbb{C}$ over $\mathbb{R}$, defined as $\langle z, w \rangle = z \bar{w}$. We'll be using the bilinear form as the distance function. The **norm form**, another important concept, is equal to $N(z) = z \bar{z}$.

In metric spaces(and more generally topology) we're concerned with open and closed sets. An **open disc** with fixed radius $r$ as a function of point $z_0 \in \mathbb{C}$ is defined as:


$$
D_r(z_0) = \{z \in \mathbb{C} : |z-z_0| < r\}
$$


In a set $\Omega$, $z_0$ is an **interior point** if there exists some open disc around it that's also contained within $\Omega$.

A set is **open** if every point in the set is an interior point. A complement of an open set is a **closed set**. A closed set contains all the limit points of the set, which are defined as the limits of convergent sequences $\\{z_n\\} \in \Omega$. The **closure** of a set is the union of $\Omega$ and its limit points, and is denoted by $\bar{\Omega}$. The **boundary** of a set $\Omega$ is the closure subtracted by all of the interior points, and is denoted $\partial \Omega$.

An open set is considered **connected** if it's not possible to find disjoint non-empty sets $\Omega_1, \Omega_2$ such that $\Omega = \Omega_1 \cup \Omega_2$.

## Sequential compactness

A set $\Omega$ is **sequentially compact** if every sequence has a convergent subsequence. An **$\epsilon$-net** is a union of balls with some fixed $\epsilon > 0$ centered around a subset of points in $\Omega$ s.t. $\bigcup B_\epsilon(x_\alpha) = \Omega$. A set is **totally bounded** if it has a finite $\epsilon$-net for every $\epsilon$. An **open covering** of a set $\Omega$ is a set of sets $\\{U_\alpha \\}$ such that $\Omega \subset \bigcup U_\alpha$. The set of open covers doesn't need to be countable. 

**Theorem: A metric space $\Omega$ is sequentially compact $\iff$ complete and totally bounded.**

**"Proof"**: In the $\implies$ direction, since every cauchy sequence is a sequence itself, it must have a convergent subsequence. Cauchy sequences are themselves convergent if they have a convergent subsequence. Therefore, cauchy sequences converge and $\Omega$ is complete. Suppose $\Omega$ is not totally bounded, then there exists some $\epsilon > 0$ such no finite $\epsilon$-net exists. Then we can create a sequence such that no subsequences can converge, by picking points outside of the $\epsilon$ ball of any of the previous points (at any point in building the sequence, we have $n$ points, and we know that the union of these $n$ points' balls is not equal to the entire set $\Omega$, so there's some point out there for us to choose). We know these points will continue to exist because there is no finite $\epsilon$-net. This contradicts that $\Omega$ is sequentially compact.

In the $\impliedby$direction, consider an arbitrary sequence. If we have finite balls of size $\epsilon$ covering the entire set (bounded), then by pigeonhole principle there must be infinitely many points of the sequence in at least one of these balls. This subset is another totally bounded set itself, but restricted to a diameter of $\epsilon$. We then state that it must have a $\epsilon/2$-net, and continue the procedure such that we have infinite nested subsets of decreasing diameter. By picking a single point in each of these subsets we've created a cauchy subsequence of the original. Cauchy sequences converge in complete metric spaces, and since our sequence is arbitrary we are done. $\blacksquare$ 

**Lemma: (Lebesgue covering lemma) Given an open cover $\\{G_\alpha\\}$ of a sequentially compact metric space $\Omega$, there exists $\delta > 0$ such that the balls of radius $\delta$ centered around any point $x \in \Omega$ is a subset of one of the open sets $G_\alpha$.**

**"Proof":** If this is false, then we can create a sequence $\\{x_n\\}$ such that the $k$-th element is the center of a ball of radius $1/k$ and it's not a subset of any $G_\alpha$. If there is no such $\delta$, we should always be able to find such $x_k$ to create an infinite sequence. Since this is a sequentially compact space, there exists a subsequence of $\\{x_n\\}$ that will converge to a point $x \in \Omega$, which must also belong to some open set $G_\alpha$. Since the point belongs to an open set, it's an interior point, which means $\exists \epsilon > 0$ such that $B_\epsilon(x) \subset G_\alpha$. We said the ball $B_{1/n}(x_n)$ is not a subset of any $G_\alpha$, but eventually elements belonging to the infinite subsequence will be close enough to $x$ and the ball will be close enough such that $B_{1/n}(x_n) \subset B_\epsilon(x) \subset G_\alpha$, which is a contradiction. $\blacksquare$ 

---

## Compactness

We call $\Omega$ **compact** if every open covering of $\Omega$ has a finite subcovering. Additionally, the **Heine-Borel** theorem states that a set in $\mathbb{R}^n$ is compact $\iff$ it's closed and bounded. An example of a compact set is $\\\{\frac{1}{n} : {n \in \mathbb{N}}\\} \cup \\{0\\}$. It's bounded in the range $[0, 1]$, and it's closed, because the limit point is $0$.

**Theorem: In metric spaces $\Omega$ is compact $\iff$ every sequence $\\{z_n\\}$ in $\Omega$ has a subsequence that converges to a point in $\Omega$ (sequentially compact).**

**"Proof"**: In the $\implies$ direction, if we assume that the set $\Omega$ is not sequentially compact, then there exists a pathological sequence $\\{x_n\\}$ such that there are no convergent subsequences. If that's the case, then for any point $x \in \Omega$, there is an adequately small radius $\epsilon > 0$ such that no point in $x_n$ is in it (other than $x$ itself, if it's in the sequence). Then we have a collection of open balls that form an open covering of the compact set. It's obvious to see that we can't use any finite subset of this collection to form the whole set, as each set only contains a single point in the space.

In the $\impliedby$ direction, let $\\{G_\alpha\\}$ be an arbitrary open covering for a sequentially compact $\Omega$. Lebesgue covering lemma states that $\exists \delta \forall x \exists \alpha$ such that $ B_\delta(x) \subset G_\alpha$. Being sequentially compact means it's totally bounded, which means there exists finite $\epsilon$-nets for any $\epsilon$. If we pick $\epsilon$ to be $\delta$, then that means there are finitely many of these $B_\delta(x_i)$'s that cover the entire set. If we look at these balls $\\{B_\delta(x_i) : i = 1, 2,...,n\\}$ which individually satisfy $B_\delta(x_i) \subset G_{\alpha(i)}$, we can just pick the finite subset of $G_\alpha$'s associated with the balls to create a finite covering of $\Omega$. Since $\\{G_\alpha\\}$ is arbitrary we are done. $\blacksquare$ 

It's important to note that sequential compactness is not equal to compactness in all scenarios. It's only true in the case of metric spaces and not general topological spaces.

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>