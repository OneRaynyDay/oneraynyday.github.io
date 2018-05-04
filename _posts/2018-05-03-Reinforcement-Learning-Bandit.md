---
published: true
title: Reinforcement Learning - Bandit Problems
use_math: true
category: ML
layout: default
---

To begin, we should note that **bandit problems** are a subset of **tabular solution methods**. The reason it's called tabular is because we can fit all the possible states into a table. The tables tell us everything we need to know about the state of the problem, and so we can often find the exact solution to the problem that's posed.

# $k$-armed Bandit Problem

> _One-armed bandit_: a slot machine operated by pulling a long handle at the side.

We are faced with $k$ different actions. Each action gives you an amount of money, sampled from a distribution conditioned on the action. Each time step, you pick an action. You usually end with $T$ time steps. **How do you maximize your gain?**

For some time step $t$, we have an action, $A_t$, and a reward of the action, $R_t$. We denote the **value\* of an arbitrary action $a$** as $q_*(a)$:

$$q_*(a) = E[R_t|A_t = a]$$

What does this mean in english? It means "the value of an action $a$ is the expected value of the reward of that action(at any time)."

After reading that sentence 3-4 times, it makes sense right? If you knew that doing 1 action will give you the greatest expected value, then you abuse the hell out of it to max your gains.

Now we obviously want $q_*(a)$. It's hard to get, though. $Q_t(a)$ is an estimate of $q_*(a)$ at time $t$. A desirable property is that:

$$lim_{t\to \infty}Q_t(a) = q_*(a)$$

So how do we make the $Q_t(a)$?

---

_value*_: in this case, it's different than the concept of rewards. Value is the long run metric, meanwhile reward is the immediate metric. If you got hooked on heroin, it would be awesome rewards for the first hour, but it would be terrible value.



