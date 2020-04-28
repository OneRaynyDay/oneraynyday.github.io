---
published: true
title: Queueing Theory - Part 1 - WIP
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

## Probability Review

TODO

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

Then, plugging into the second axiom of probability (probability of the whole sample space sums to 1, sort of), we solve for $P_0$:

$$\sum_{i=0}^sP_i = \sum_{i=0}^s \frac{(\lambda \tau)^i}{i!}P_0 = P_0 \sum_{i=0}^s \frac{(\lambda \tau)^i}{i!} = 1 \\ \implies P_0 = (\sum_{i=0}^s \frac{(\lambda \tau)^i}{i!})^{-1}$$

Solving for the general $P_j$, we have

$$P_j = \frac{(\lambda \tau)^j}{j!} (\sum_{i=0}^s \frac{(\lambda \tau)^i}{i!})^{-1}$$

So now we know what the long run proportion of time the store has $j$ people in it. We can then model the congestion of this supermarket and do more cool things with it.

## Single Server System - How many customers are blocked?

In queueing theory, the most basic system consists of a **single server** with arrival of customers modeled as an i.i.d process. You can imagine this as a sequence of independent **cycles**, defined by a period of inactivity (the store has noone coming in), and a period of activity (the store is serving a customer, with possibly more customers arriving during the interim period). To be precise, at the end of the period of activity, there should be no customers currently in the queue. *Intuitively, it makes sense that each cycle should be independent, as we return to 0 customers and the arrival of customers are not independently distributed.* 

Suppose the number of customers that arrive **during the busy period** in an arbitrary cycle is a random variable $N$, then below is an interesting result:

$$
P(\text{customer is blocked}) = \frac{E(N)}{1+E(N)}
$$

Intuitively, this makes sense. If we consider each cycle, the number of customers is $E(N) + 1$ since the first customer arrived during the inactive period, and the number of blocked customers are $E(N)$ because each one had to wait for the previous customer to finish being served. However, let's prove this:

First, let's use the law of total probability here to make this a little easier. We define a **customer to be of type $i$** if they arrived when $i$ people arrived in a cycle. Note that the first customer who arrived and was served immediately is also a customer of type $i$ in this event.

$$
P(\text{customer is blocked}) = \sum_{i=1}^\infty P(\text{customer is blocked} | \text{customer is of type i})P(\text{customer is of type i})
$$

Well... This is good and all, but how do we get $P(\text{customer is of type i})$ to express $E(N)$? For there to be a customer of type $i$ we must have $N = i-1$, since there must be $i-1$ waiting customers in this cycle. We thus have the following probability:

$$
P(\text{customer is of type i}) \\
= \frac{i P(N=i-1)}{\sum_{k=1}^\infty kP(N=k-1)} \\
= \frac{i P(N=i-1)}{\sum_{k=1}^\infty((k-1) + 1)P(N=k-1)}\\
= \frac{i P(N=i-1)}{E(N) + \sum_{k=0}^\infty P(N=k)} \\
= \frac{i P(N=i-1)}{E(N) + 1}
$$

*NOTE: If you're intimidated by the denominator, this is setup from the [ball-and-urn](https://en.wikipedia.org/wiki/Urn_problem) type model where the i-th urn is associated with* $\{N=i\}$, *and the probability that a random ball chosen from the population belongs to the i-th urn is equivalent to* $\{\text{customer arrived when N=i}\}$. *The ball and urn set-up is added in the appendix*.

We know that $P(\text{customer is blocked} \| \text{customer is of type i}) = \frac{i-1}{i}$ since there is a $\frac{i-1}{i}$ chance that this customer is not the first customer (which is the only one not blocked).

When we substitute these quantities into the above total probability equation, we get:

$$
\sum_{i=1}^\infty P(\text{customer is blocked} | \text{customer is of type i})P(\text{customer is of type i}) \\
= \sum_{i=1}^\infty (\frac{i-1}{i})(\frac{i P(N=i-1)}{E(N) + 1}) \\
= \frac{\sum_{i=0} i P(N=i)}{E(N) + 1} \\ = \frac{E(N)}{E(N)+1}
$$

# Appendix

## Ball-and-Urn Reference

This is a problem from intro to probability classes. Suppose we have finite or countably many urns, indexed by $1,2,...$ where each $i$th urn includes $n_i$ balls. We want to know whats the probability of getting a ball from the $i$th urn w.r.t to the probability  of picking the $i$th urn. We denote $P(Y=j)$ as the probability of the $j$th urn being chosen from the population of urns, and $P(X=j)$ as the probability that a ball from the $j$th urn is chosen from the population of balls. We have the following result:

$$
P(X=j) = \frac{n_j P(Y=j)}{\sum_i n_i P(Y=i)}
$$

Since in our queueing theory example, $n_j \approx j$ ($\pm$ some constants), we were able to express the bottom sum as $E(Y) + C$.