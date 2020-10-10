---
published: true
title: Computability Theory - Post's Problem & Priority Argument (Part 2)
use_math: true
category: math
layout: default

---

# Table of Contents

* TOC
{:toc}


*wip*

# Recap

*Note: If you haven't read part 1, this entire blog will make no sense to you.*

Last time, we set up the notion of Turing reducibility and oracle machines, and asked the question, *does there exist recursive enumerable sets $$A,B​$$ such that $$A \not\leq_T B​$$ and $$B \not\leq_T A​$$?* , where $$\leq_T​$$ is Turing reducibility. Emil Post, one of the leading figures in computability theory, asked this question. He was never able to answer his own question, since he died early of electroshock therapy for depression. Two mathematicians, Friedberg and Muchnik later solved the problem in the same method, which we now call **Priority Argument**. We have set up the intuition for how the priority method may work in part 1, but we failed to show computability of the two sets, since in every iterative stage we relied on the following predicate:

$$
\forall x, \phi_e^A(x)\uparrow
$$

Which, as we explained, is definitely **not computable**. Then how do we get around this?

# Intuition

We can never know whether a particular $$\phi_e^A$$ will stall on all inputs, but during the time we're waiting for it to halt, we can continue the next few stages in parallel, assuming that it's not going to halt on anything. Then there are two cases associated with this strategy:

1. $$\phi_e^A(x) \uparrow \forall x$$, then what we have assumed for this program is fine.
2. $$\exists x, \phi_e^A(x) \downarrow$$, then what we have assumed is false, but **this will only occur once**. Then, we can tell all programs we're currently running, i.e. $$\phi_{e+1}^A, \phi_{e+2}^A, ...$$ to restart, with our new, corrected initial segment $$x_e$$ (this was covered in the construction from part 1). The concept of having to restart a program, but finitely many times is the concept of **finite injury**, and the concept of all programs after $$\phi_e^A$$ being able to be restarted at will by the strategy dictating $$\phi_e^A$$ is the concept of **priority**.

The description above may seem ambiguous, so stick around for the construction.

# Construction

We formally define **strategy** below as some algorithm that tries to meet our requirements. Our requirements in part 1 are exactly the same here (either #1 or #2):

1. $$\phi_e^A(x)\uparrow \forall x$$, and $$g_B(x_{e-1}+1)$$ can be anything (recall $$g_B$$ is a characteristic function so it is total).
2. $$\exists x_e > x_{e-1}, \phi_e^A(x_e) \downarrow \neq g_b(x_e)$$.

Then we implement the following strategies $$S_{2e}$$ for the $$e$$-th step of making $$\phi_e^A(x_e) \neq g_B(x_e)$$, and the strategies $$S_{2e+1}$$ for $$\phi_e^B(y_e) \neq g_A(y_e)$$. We say the strategy $$S_i$$ has **higher priority** than $$S_j$$ iff $$i < j$$. This means that $$S_i$$ has the power to restart $$S_j$$ whenever it wants to, with a new initial segment (Notice how the strategies for $$g_A$$ and $$g_B$$ can restart each other). We say that an initial segment up to $$x_e$$ is **restricted** for strategies $$S_i, i > 2e$$, since it is not allowed to change the initial segment, as its priority is too low. However, any strategy $$S_k, k \leq 2e$$ is allowed to modify the initial segment up to $$x_e$$. A strategy $$S_{2e}$$ is **allowed to act** when $$\phi_e^A(x_e)\downarrow$$ for some $$x_e$$ - this means our initial guess that the program $$\phi_e^A$$ loops on all inputs is false, and so we must modify the initial segment $$x_e$$. When the strategy acts, it restarts all strategies $$S_j \forall j > 2e$$, which restart to obey the newly updated, restricted initial segment.

Our algorithm can be best explained in pseudocode, so let's describe it:

```python
# We denote phi_e^A as e-th A-program here, similarly for B
# We also denote x_e as the initial segment index for e-th A-program, 
# 	and similarly for y_e for B.
for I=0,1,2,...
	e = I/2
	if I % 2 == 0: # I is even
        # run strategy for A
        for i=1,2,...,e-1
        	Run i-th A-program on x_i,x_i + 1,...,x_i + e for e steps
        if e > 0:
            x_e = x_{e-1} + 1
        else:
            x_e = 0
        Run e-th A-program on x_e for e steps.
        
        # When strategy 2i acts, x_i' = x_i+j, and all k-th programs currently running
        # where k > i, are restarted with the indices x_k = x_i' + (k-i).
        if any highest priority i-th A-program terminated on some x_i + j, then allow strategy 2i to act.
        
    else: # I is odd
        # run strategy for B
        for i=1,2,...,e-1
        	Run i-th B-program on y_i for e steps
        if e > 0:
            y_e = y_{e-1} + 1
        else:
            y_e = 0
        Run e-th A-program on y_e for e steps.
        
        if any i-th B-programs terminated, then allow strategy 2i+1 to act.
```

What we are doing here is **computably guessing whether the predicate $$\phi_e^A(x)\uparrow \forall x$$ is true**. If it fails, we know it will fail finitely many times, and our program will eventually reach the result from part 1.  

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
