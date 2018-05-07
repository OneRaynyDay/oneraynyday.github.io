---
published: true
title: Reinforcement Learning - Markov Decision Process
use_math: true
category: ML
layout: default
---

Previously, we discussed **$k​$-armed bandits**, and algorithms to find the optimal action-value function $q_*(a)​$. 

Once we found $q_\*(a)$, we could just choose $argmax_a q_\*(a)$ all the time to reach the optimal solution.

We now go one more step further, and add a _context_ to our reinforcement learning problem. Context in this case, means that we have a different optimal action-value function for every state:

$$q_*(a,s) = E(R_t|A_t=a, S_t=s)$$

This situation, where we have different states, and actions associated with the states to yield rewards, is called a **Markov Decision Process(MDP)**.

# Formalizations

## The Agent-Environment Interface

We need to establish some notation and terminology here:

1. The decision maker is called the **agent**.
2. The decision maker's interactions is with the **environment**, which give rewards.

For some time steps $t = 0,1,2,3,…$, we have associated with it, a state and an action $S_t \in \mathcal{S}, A_t \in \mathcal{A(s)}$. The reason actions is a function of state is we might have different permitted actions per state. If actions are the same in all states, then sometimes people abbreviate it as $\mathcal{A}$. With the associated action $A_t$, we receive a reward: $R_{t+1} \in \mathcal{R} \subset \Re$.

We thus have a sequence that looks like:

$$S_0 \to A_0 \to R_1 \to S_1 \to A_1 \to R_2 \to …$$

In a finite MDP:

$|\mathcal{S}| < \infty, |\mathcal{A}| < \infty, |\mathcal{R}| < \infty$. 

$(R_t, S_t)_{t\geq 0}$ is a markov chain here. A **markov chain**, under discrete sigma algebra, is defined as:

$$P(X_n = x_n|X_{n-1}=x_{n-1}, X_{n-2}=x_{n-2}, X_{n-3}=x_{n-3},…,X_0=x_0) = P(X_n = x_n|X_{n-1}=x_{n-1})$$

Or in other words, $X_{n+1}$ , the future state, is dependent only on $X_{n}$, the present state. This is often called the **markov property**. 

Thus, with the assumption, we can describe the entire dynamics of a finite MDP given:

$$P(S_t = s', R_t = r | S_{t-1} = s, A_{t-1}=a) \forall s',r,s,a$$

We can marginalize $R_t$ and get:

$$P(S_t=s'|S_{t-1}=s, A_{t-1}=a)$$

which tells us the **state-transition probabilities**.

The expected reward for a state-action pair can be defined as:

$$r(s,a) = E(R_t|S_{t-1}=s, A_{t-1}=a) = \sum_{r \in \mathcal{R}}r\sum_{s'\in\mathcal{S}}p(s',r|s,a)$$

## Returns and Episodes

In general, if the agent has a finite lifespan, say $T$, then we want to maximize the reward sequence $G_t$:

$$G_t = R_{t+1}+R_{t+2}+R_{t+3}+…+R_T$$

Sometimes, $T$ can be a random variable. 

However, many times our agent has an infinite lifespan, or we're optimizing it to go to $\infty$(think of maximizing the time a robot is standing up). What do we do for that? We can re-formulate our problem using the idea of **discounting**:

$$G_t = \sum_{k=0}^\infty \gamma^k R_{t+k+1}$$

This pretty much says, "the immediate rewards are better than the rewards later on, and the rewards way later on approaches 0". And this discounting idea can apply to finite lifespan episodes as well, because we can have a final absorbing state $S_0$ where the rewards associated with any action in that state is 0.

## Policies and Value Functions

We want to know _how good it is to be in a specific state_, and _how good it is to take an action in a given state_. 

A **policy** is a mapping from states to probabilities of selecting an action. If an agent is following policy $\pi$, then he will take action $a$ at state $s$ with probability $\pi(a\vert s)$.

A **value** of a state $s$, under a policy $\pi$, is the expected return starting from $s$ if we take the policy's suggested actions. It is denoted by $v_\pi(s)$, and is called a _state-value function_.

$$v_{\pi}(s) = E_\pi(G_t|S_t=s) = E_\pi(\sum_k^\infty\gamma^kR_{t+k+1} | S_t=s)$$

If we fix the action that we take at state $S_t = s$, then we would get an _action-value function_ $q_\pi(s,a)$.

$$q_\pi(s,a) = E_\pi(G_t|S_t=s,A_t=a)$$

