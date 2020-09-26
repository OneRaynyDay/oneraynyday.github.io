---
published: true
title: Reinforcement Learning - Introduction
category: ML
layout: default
---

_It's been a while since I wrote anything to the blog_, and it's partially because life has been going real fast that I haven't had time to really think about jotting down my thoughts. I've been chasing a pineapple-pizza-eating, hot-chocolate-drinking someone so that's another excuse :-).

Some habits die hard, and I always think about writing another blog, to learn something new and really remember it.

This time, it'll be about **reinforcement learning**! It's been a topic at the top of my head, one of those "magical" things that aren't really so special once you look at it closely. I'll be closely following the book: **"Reinforcement Learning: An Introduction" by R.S.Sutton and Andrew Barto**.

As with all "introduction to" books, this book runs the whole gamut, and is totally not just an introduction.

We will tackle this from the most simple models and get more complex from there. I've been wanting to learn more about the following(in increasing difficulty):

## Tabular Solution Methods

This is more of the discrete domain space, faster solve, less gradient descent-y, less convex optimization type algorithms:

1. Multi-armed Bandit Problems
2. Finite MDP (Have some background on this, but review's nice)
3. DP methods (Have some introductory background on this)
4. Monte Carlo Methods (Sounds fun!)
5. Temporal-Difference Learning (Never heard of it)
6. n-step Bootstrapping (eh, bootstrapping is in every book)

## Approxixmate Solution Methods

This is more of the "sexy", gradient-based, eigenvalue, continuous domain, more convex(or even non-convex) optimization type algorithms:

1. On-policy prediction (SGD/linear methods/kernels/ANN's)
2. Off-policy prediction (No idea what this is)
3. Elligibility Traces (No idea what this is)
4. Policy gradient (very fancy, new, sexy formulation of an old method)

And if we're feeling frisky:

## Frontiers

1. General value functions
2. Designing reward signals
3. AlphaGo

Expect to be seeing more soon! Whoever is reading this should keep me accountable. I'll try to get one thing done every one or two weeks.

In the end, I'll try my best to write some basic library like last time, but this time for RL.

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
