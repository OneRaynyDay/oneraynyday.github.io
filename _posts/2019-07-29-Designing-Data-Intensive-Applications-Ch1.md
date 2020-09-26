---
published: true
title: Designing Data Intensive Applications - Chapter 1
use_math: true
category: dev
layout: default
---

# Table of Contents

* TOC
{:toc}
# What Do We Care About?

## Reliability

**Definition**: Working correctly under faults and errors.

We need reliability in a system because _users can be stupid and malicious and make mistakes_. We want the system to be fault-tolerant in these conditions.



### Aside: Fault vs. Failure

A **fault** is defined as a component of a system not behaving as it is supposed to. A **failure** is defined as system-wide inability to perform a function. Sometimes, faults in fault-tolerant systems are triggered deliberately so that it is continuously tested. 

_TL;DR:_ Faults can happen, but we need to respond to it so that no failures happen.



### Aside: Types of Faults

A **hardware fault** is when a machine dies, and is largely random. This is often solved by redundancy via RAIDs, dual power supplies, etc. You can actually write software to increase hardware fault-tolerance.

A **software fault** is often logical. This is somewhat harder because writing software to find bugs in software is non-computable mathematically (it's not even Turing-recognizable). The only thing we can do is design the system so that errors are propagated up to the developer as soon as possible.

_TL;DR:_ Hardware faults are random and easier to solve than software faults, which are often due to bugs in code.

## Scalability

**Definition**: How well a system can deal with growing complexity or load.

Load is a very ambiguous concept. It's simply a metric of things that often add complexity to a system. For example, requests per second, hit rate on a cache, amount of data in a database.

When we increase load, we either want to keep the system the same, and see how the performance is affected, or vice versa. For example, **throughput** is the amount of things done by a system, and **response time** is the time required by the system to do a thing. Similar to response time, **latency** is the time spent waiting for a request to be handled. Latency is factored into response time. Looking at throughput and response time are two sides of the same dice.

_TL;DR_: To analyze load, you want to analyze how fast the system processes one thing, or how many things a system can process in a given time interval.

### Aside: Statistical Analysis on Response Time

Once enough response time metrics are obtained, we retrieve a discrete distribution. Often people report the **average response time**, but it doesn't really take into account how many users are actually experiencing the delay. We often care about this because the users experiencing the highest delay are the ones using the system the most. They often make us the most money.

Instead, we should use **percentiles**. The 99th percentile are the slowest 1% response times, often due to a large latency. Often, percentiles are used in **SLA (service level agreements)**. 

_TL;DR:_ Use percentiles, not the average.

### Aside: How to Deal With Increasing Load

In order to deal with increasing load, one can **scale up/vertically** by moving to a more powerful machine, or **scale out/horizontally** by adding more machines. Obviously, the power of a machine and its price is often nonlinear, and we want to be cheap. However, not all systems can be scaled out efficiently. There is a tradeoff dependent on the architecture. **Elastic scaling** is automatically scaling when detecting increasing load. This can be vertical or horizontal but most often it is horizontal.

_TL;DR_: To scale, throw more machines at the problem or buy a beefier one.

## Maintainability

**Definition**: How well the system is able to be maintained and improved.

Noone likes legacy systems because reading other people's code is hard. Most of the cost of running a system is not during development but during maintenance by an operations team. Let's save some money:

- **Operability**: You want the operations team to easily maintain the system.
- **Simplicity**: You want new engineers to not run away when they try to understand the system.
- **Evolvability**: Make it easy to add changes to the system.

_TL;DR_: Design a system with maintainance and extensibility in mind.

### Aside: What Operation Teams Do

Operation teams write a lot of tooling around monitoring, failure investigations, security patching, etc. They are one of the most important teams, but are often overlooked because they wipe the subpar developers' asses. They often do migration, maintenance, config management, deployment, documentation, etc. It's a lot of hard work.

_TL;DR_: Operation teams are unsung heroes who make sure the system is well maintained.

### Aside: How to KISS

By KISS, I meant Keep It Simple, Stupid. When a project gets large, uncontrolled complexity grows roughly quadratically (There are $$N^2$$ edges in a complete simple undirected graph with $$N$$ nodes). We want to keep the complexity growth as linear as possible when new components are added.

Sometimes it's the subpar developer's fault, but sometimes it's the customer's fault. Often times, stupid anti-patterns emerge where the users do something they're not supposed to and future versions of projects need to keep backwards compatibility. This accidental complexity is there to stay, and that sucks.

To remove complexity, we use **abstractions**. Basically hide a lot of implementation details under a simple interface. This is surprisingly hard.

_TL;DR_: Don't make things too complicated, and prevent users from doing stupid shit by hiding things from them.

### Aside: How to Evolve

This is more of a engineering design process kind of issue. The development team needs to be receptive to constant change due to regulations, scalability concerns, etc. **Agile** is a complicated development pattern that focuses on iterative development and frequent introspections in order to change specifications on the fly. 

_TL;DR_: Agile is one method to allow for frequently changing specifications and evolution of a project.





<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
