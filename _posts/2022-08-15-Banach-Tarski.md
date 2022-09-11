---
published: true
title: Probability Tidbits 1 - Banach Tarski
use_math: true
category: math
layout: default
---

# Probability Tidbits 1 - Banach Tarski
**Not all sets are measurable.** Within $[0,1]$ there exists unmeasurable sets. We'll construct one here. First define an equivalence class $x \sim y \iff x - y \in \mathbb{Q}$. The set $[0,1]$ contains many such equivalence classes(which by definition are disjoint from each other). By axiom of choice, we can choose a representative from each equivalence class. Let $A$ be a set containing the representatives. We construct a disjoint cover:

$$
[0, 1] \subset \bigcup_{r \in \mathbb{Q}_{[-1,1]}}\{a+r: a \in A\} := U
$$

Where $$\mathbb{Q}_{[-1,1]} := \{x: x \in \mathbb{Q}, -1 \leq x \leq 1\}$$. Denote each $$X_r := \{a+r:a \in A\}$$. $U$ covers $[0,1]$ because $$\forall x \in [0,1], x \sim a$$ for some $a \in A$ by definition, and so $$x - a \in \mathbb{Q}_{[-1,1]}$$. For $r \neq s$, $X_r \cap X_s = \emptyset$ because suppose they're not disjoint, then: 

$$
\begin{align}
x \in X_r \cap X_s \\
\implies \exists a_r, a_s \in A \text{ s.t. } a_r+r = a_s + s = x \\
\implies a_r - a_s = s-r \in \mathbb{Q} \\
\implies a_r \sim a_s \\
\implies a_r = a_s \\
\implies r = s
\end{align}
$$

Which is a contradiction. Also, $U \subset [-1, 2]$ because 

$$r \geq -1, a \geq 0 \implies a+r \geq -1 \forall a \in A, r \in \mathbb{Q}_{[-1,1]}$$ 

and 

$$r \leq 1, a \leq 1 \implies a+r \leq 2 \forall a \in A, r \in \mathbb{Q}_{[-1,1]}$$

Because lebesgue measures are translation invariant:

$$
\begin{align}
\mu(X_r) = \mu(A) \quad \forall r \in \mathbb{Q}_{[-1,1]} \\
\implies \mu([0,1]) \leq \mu(\bigcup_{r \in \mathbb{Q}_{[-1,1]}} X_r) \leq \mu([-1,2]) \\
1 \leq \sum_{r \in \mathbb{Q}_{[-1,1]}} \mu(A) \leq 3
\end{align}
$$
which is impossible.

Knowing that some sets aren't measurable is important - it means we can't assign a probability to specific sets of outcomes. We need a more rigorous definition for probability theory, and that's where measure theory comes in.

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
