---
published: true
title: Reinforcement Learning - Temporal Difference Learning (Q-Learning & SARSA)
use_math: true
category: ML
layout: default
---

# Table of Contents

* TOC
{:toc}

Temporal difference learning is one of the most central concepts to reinforcement learning. It is a combination of Monte Carlo ideas [todo link], and dynamic programming [todo link] as we had previously discussed.

## Review of Monte Carlo

In Monte Carlo, we have an update rule to capture the expected rewards $V$ under a given policy $\pi$. Usually, we simulate entire episodes and get the total reward, and attribute each action in the action sequence to the reward value. Recall we had two different ways of Monte Carlo simulation - **every-visit** and **first-visit**.

Given some state sequence $S_j = (s_0, s_1, ...)$, and action sequence $A_j = (a_0, a_1, ...)$ belonging to some episode $j$, **first-visit MC** will perform the following:

$$
V(s) = \frac{1}{N}\sum_{i:s \in S_i}^N G^i_{min_j\{s_j | s_j = s\}}
$$

For some reward $G^i_t$ which is the (possibly discounted) sum of subsequent rewards in episode $i$ after time $t$. In other words, if a state occurs once, or twice, or fifty times throughout an episode, we only count the first occurence of the state. and average all of the reward values for $V_s$. For **every-visit MC**, we have:

$$
\mathcal{G}^i_s = \sum_{j:s_j = s, s_j \in S_i \forall S_i} G^i_j \\
V(s) = \frac{1}{\#s} \sum_i^M \mathcal{G}^i_s
$$

where #s is the total number of times we saw state $s$ throughout all episodes. In other words, if a state occurs more than once, we will add those occurences into our value approximation and average all the rewards values to get $V(s)$.

In many cases, $V(s)$ is stationary, and both methods (first-visit and every-visit) will converge to the optimal answer. However, in a nonstationary environment, we do not want our "update magnitude" to approach 0. The below is an update rule suitable for a nonstationary, every-visit MC:

$$
V(S_t) = V(S_t) + \alpha [ G_t - V(S_t) ] & \text{(0)}
$$

for some constant $alpha > 0$.

# Temporal Difference Prediction

In the above equation, we use the quantity $G_t$, which is the rewards for the rest of the episode following time $t$, possibly discounted by some factor $\gamma$. This quantity _is only known once the episode ends. For TD, this is no longer necessary!_

How does TD methods do this? They only need to look at one step ahead, because their approximation is slightly less precise and less sample-based. Instead of $G_t$, they use:

$$
R_{t+1} + \gamma V(S_{t+1})
$$

Recall that the equation for $G_t$ is:

$$
G_t = \sum_{k=t+1} \gamma^{k-t-1}R_k = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + ...
$$

And the equation for $V(S_t)$ is:

$$
V(S_t) = E[G_t]
$$

And so we're sort of "approximating" the rest of the trajectories:

$$
V(S_{t+1}) = E[R_{t+2} + \gamma R_{t+3} + ...] \approx R_{t+2} + \gamma R_{t+3} + ...
$$

So yes, it is less _precise_, but in return, _we are able to learn in the middle of an episode_. The resulting update equation for $V(S_t)$ in the analog of $\text{eq (0)}$ is:

$$
V(S_t) = V(S_t) + \alpha [ R_{t+1} + \gamma V(S_{t+1}) - V(S_t) ]
$$

In this case, since we're only looking one step ahead, it is called $TD(0)$. Here's some pseudocode logic:

```
pi = init_pi()
V = init_V()
for i in range(NUM_ITER):
    s = init_state()
    episode = init_episode(s)
    while episode:
        prev_s = s
        action = get_action(pi, V)
        reward, s = episode.run(action)
        V(prev_s) += alpha * (reward + gamma * V(s) - V(prev_s))
```

$TD(0)$ is considered a _bootstrapping_ method because it uses previous approximations, rather than completely using the entire trajectory's reward.

#### Similarity between TD and DP

Recall that dynamic programming approaches to solve reinforcement learning problems uses the Bellman backup operator. This allows efficient incremental computation of $V(s)$, and we previously proved that $lim_{k \to \inf} v_k(s) = v_\pi(s)$. In this way, we are effectively getting more and more accurate approximations of $v$.

In a similar nature, _our approximation of $V(S_t)$ in the equation above is also iterative, and bootstraps off of the previous estimation_, because $V(S_t)$ as a function is not known.

#### Similarity between TD and Monte Carlo

Recall that Monte Carlo requires us to estimate the expected value, because we're taking samples of episodes and trying to approximate the underlying expected value from each action. In this way, TD acts like a every-visit MC due to its $V(S_t)$ being an approximation to the expected value of $G_t$.

Thus, TD and Monte Carlo both _approximate $V(S_t)$ via sampling because the expectation is not known._ However, Monte Carlo does not attempt to use previous estimations of $V$ in its convergence.







