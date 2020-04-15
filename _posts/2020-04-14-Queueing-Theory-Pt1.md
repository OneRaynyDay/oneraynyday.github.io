---
published: false
title: Queueing Theory - Part 1
use_math: true
category: math
layout: default
---

Apologies for the long delay in posts! I've been caught up with some busy work and only recently did I remember to revisit my blog. This time around we'll be doing some reading for **queueing theory**, which is one of the most relevant mathematical fields for computer science. For the sake of this blog, I will not get into measure theoretic tools required to answer particularly complicated examples. In addition, please bear with me when there is inevitable mathematical rigor lost by applying real life examples and intuition (without this turning into a 10 page tangent on every small detail).

# What is queueing theory?

Instead of giving you a bland introduction, let me illustrate a picture that's relevant to the times.

Imagine you're lining up for a supermarket. Due to COVID-19, the store only allows $N$ number of visitors inside at once. As the manager of this supermarket, you probably wish you knew some queueing theory. This helps you answer questions like:

1. Can we assume the customers will be waiting indefinitely outside, or do they see that there's a line and leave immediately? Or is it something in between?
2. How many employees/self-checkout counters do I need to make the waiting time for people outside be within acceptable levels?
3. How many people should we let in for some fixed period of time? As in, what is the biggest $N$ we can permit (without getting everyone sick)?

## Defining basic terms

In the above example, we define the **input process** as the distribution of the lengths of time between consecutive customer arrivals. The **service mechanism** is the number of employees we have, and the number of self-checkout counters. The **queueing discipline** is the behavior of blocked customers - as in whether they would leave immediately, or if they would wait for a period of time before realizing the toilet paper is all gone and they're ready to go home.

This field of mathematics employs a very important concept, called **conservation of flow**. The CS undergrads would recall the problem *max flow/min cut*, and how it uses the idea of conservation of flow as well. In this context, we assume in the long run the supermarket will achieve an equilibrium with the number of people inside the store at any point in time. This means that the number of customers coming in will be the same as the number of customers going out.

## Simple queueing system problem

Let's say we have a ***poisson process*** occurring with the above supermarket example. By this I mean the following:

1. The average number of customers coming in over some fixed unit time $T$ is $\lambda$. 
2. The store can fit max $s$ people at once.
3. People leave the store in $\tau$ time on average.
4. The store lets only one person in at a time.
5. Each customer is independent of the other (they enter and leave the store with no correlatio to other shoppers).

Then with this model, *we want to know in the long run the proportion of the time the store has $j \in \{0,1,...,s\}$ people in it*, denoted $P_0, P_1, ... , P_s$(you can also interpret this as the probability that the store has $j$ people in it at any point in time). We also denote the state when the store has $j$ people in it as $E_j$.

Let's break this into two cases:

### Case 1: Increasing number of customers, $E_j \to E_{j+1}$

In this case, for $j \in \{0,1,...,s-1\}$, we have the rate at which $E_j \to E_{j+1}$ as $\lambda P_j$ in unit time $T$, because at any time a customer arrives, there is a $P_j$ chance we are at $E_j$. For $j = s$, it is impossible for us to exceed the max capacity, so that rate is $0$ always.

### Case 2: Decreasing number of customers, $E_{j+1} \to E_j$

In this case, if we have $j+1$ customers in the store, each leaving after $\tau$ time on average, then we have on average $j+1$ people leaving in $\tau$ time(at this point in time). This means the rate of people leaving at $E_{j+1}$ is $\frac{j+1}{\tau}$. However, note that at any point in time, the probability of being in the state $E_{j+1}$ is $P_{j+1}$, so on average we will have $\frac{j+1}{\tau} P_{j+1}$ as the rate of which $E_{j+1} \to E_j$ when taking into account all of the other states the supermarket could be in.

### Solving system of equations

We now have a couple of equations we have to solve for $P_0, P_1, ... , P_s$. The first being the fact the sum of probabilities for all states must equal to 1, otherwise known as the **law of total probability**:

$$\sum_{i=0}^s P_i = 1$$

Then, we have that by the **conservation of flow**, in the long run the rates of $E_j \to E_{j+1}$ and $E_{j+1} \to E_j$ are the same: 

$$\lambda P_j = \frac{j+1}{\tau} P_{j+1}$$

I don't want to prove formally the induction because I'm lazy, but we can solve the recurrence relation as follows:

$$P_1 = \lambda \tau P_0 \\ P_2 = \lambda \tau P_1 = \frac{(\lambda \tau)^2}{2}P_0 \\ P_3 = ... = \frac{(\lambda \tau)^3}{3!}P_0 \\ P_j = \frac{(\lambda \tau)^j}{j!}P_0$$

Then, plugging into the law of total probability, we solve for $P_0$:

$$\sum_{i=0}^sP_i = \sum_{i=0}^s \frac{(\lambda \tau)^i}{i!}P_0 = P_0 \sum_{i=0}^s \frac{(\lambda \tau)^i}{i!} = 1 \\ \implies P_0 = (\sum_{i=0}^s \frac{(\lambda \tau)^i}{i!})^{-1}$$

Solving for the general $P_j$, we have

$$P_j = \frac{(\lambda \tau)^j}{j!} (\sum_{i=0}^s \frac{(\lambda \tau)^i}{i!})^{-1}$$

So now we know what the long run proportion of time the store has $j$ people in it. We can then model the congestion of this supermarket and do more cool things with it.