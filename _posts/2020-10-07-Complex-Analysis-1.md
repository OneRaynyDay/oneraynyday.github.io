---
published: true
title: Complex Analysis - Introduction to complex numbers and holomorphicity
use_math: true
category: math
layout: default

---

WIP.

**The complex numbers are defined as $$\mathbb{C} = \{ a+bi : a, b \in \mathbb{R}\}$$**. We look at some basic operations on the field and dive into differentiability.

# Basic Operations

Consider two complex numbers $z = a + bi$ and $w = c + di$ in the following:

- The **conjugation** of a complex number $$z$$ is defined as $$\bar{z} = a - bi$$. This is an [isomorphic](https://en.wikipedia.org/wiki/Isomorphism#:~:text=In%20mathematics%2C%20an%20isomorphism%20is,an%20isomorphism%20exists%20between%20them.) map of $$\mathbb{C} \to \mathbb{C}$$.
- A non-zero complex number can be written uniquely in **polar form** as $z = re^{i\theta}$, $r > 0, \theta \in \mathbb{R}$, and $\theta$ is called the **argument**.
- The **norm form** of a complex number $z$ is a measure of its distance from the origin: $$N(z) = a^2 + b^2$$, and is the squared distance: $$N(z) = \|z\|^2$$. The **bilinear form** of two complex numbers $$z,w$$ is defined as $$\langle z, w \rangle = Re(z\bar{w}) = \frac{(ac-b(-d))^2+(a(-d) + bc)^2}{2} = \frac{(ac+bd)^2 + (-ad + bc)^2}{2}$$.
- The **$$Re : \mathbb{C} \to \mathbb{R}$$** gives the real part of a complex number $z$ is $a$, and can be derived as $$Re(z) = \frac{z+\bar{z}}{2}$$. Similarly, the **$$Im : \mathbb{C} \to \mathbb{R}$$** part of a complex number is $$b$$ and can be derived as $$Im(z) = \frac{z - \bar{z}}{2i}$$. We have the property $\|Re(z)\|, \|Im(z)\| \leq \|z\| \leq \|Re(z)\| + \|Im(z)\|$ (draw a triangle).
- The **addition** of two complex numbers $z, w$ is defined as $z+w = (a+c) + (b+d)i$. This is a translation mapping in the real and imaginary axes.
- The **subtraction** of two complex numbers $z,w$ is defined as $z-w = (a-c) + (b-d)i$.
- The **multiplication** of two complex numbers $$z,w$$ is defined as $$zw=(a + bi)(c+di) = ac + adi + cbi + bdi^2 = (ac - bd) + (ad + bc)i$$. This is a rotation and scaling operation. 
- The **division** of two complex numbers $z,w$ is defined as $$\frac{z}{w} = \frac{a+bi}{c+di} = \frac{a (c-di)}{(c+di)(c-di)} + \frac{bi(c-di)}{(c+di)(c-di)} = \frac{(ac + bd) + (bc - ad)i}{c^2+d^2}$$. It can also be written concisely as $$z \cdot \frac{1}{w} = z \frac{\bar{w}}{N(w)} = \frac{(a+bi)(c-di)}{c^d+d^2} = \frac{(ac + bd) + (bc - ad)i}{c^2+d^2}$$.



## Complex exponentials

This deserves its own section. We can define $e^x = \sum_n^\infty \frac{x^n}{n!}$ for $x\in \mathbb{R}$. This is also true in $\mathbb{C}$. Recall the trigonometric functions:

$$
sin(x) = \sum_n^\infty (-1)^n \frac{x^{2n+1}}{(2n+1)!} \\
cos(x) = \sum_n^\infty (-1)^n \frac{x^{2n}}{(2n)!}
$$

These two look very similar, and we previous had no way of unifying these concepts together. However, plugging in $i$ into the exponential function we get an expansion:

$$
e^{ix} = \sum_n^\infty \frac{i^nx^n}{n!} = cos(x) + isin(x) 
$$

This is exactly why $e^{2\pi i} = 1$.

If we look at $e^{-ix} = cos(-x) + isin(-x) = cos(x) - isin(x)$, we can derive that $sin(x) = \frac{e^{ix} - e^{-ix}}{2i}$ and $$cos(x) = \frac{e^{ix} + e^{-ix}}{2}$$, which look very familiar to the [hyperbolic trigonometric functions](https://en.wikipedia.org/wiki/Hyperbolic_functions), which are the $sin, cos$ variants rotated-by-90 degrees in the clockwise direction in the complex plane. These forms are called the **Euler formulas**.

# Complex differentiation

Given a function $f: \mathbb{C} \to \mathbb{C}$, it is **differentiable** at $z_0$ if:

$$
\text{lim}_{h \to 0} \frac{f(z_0 + h) - f(z_0)}{h} = f'(z_0) \in \mathbb{C}
$$

Or, by delta epsilon definitions, it is differentiable at $z_0$ with derivative $f'(z_0)$ if $\forall \epsilon > 0 \exists \delta > 0$ such that for all $\|z-z_0\| < \delta$:

$$
|\frac{f(z)-f(z_0)}{z-z_0} - f'(z_0)| < \epsilon
$$

If $f$ is differentiable everywhere in its domain, then it's called **holomorphic** or **complex differentiable**. The reason we don't call it differentiable simply is because holomorphicity is _stronger_ of a property. As opposed to a real-valued function $f: \mathbb{R} \to \mathbb{R}$ which is easy to draw and visualize, $f: \mathbb{C} \to \mathbb{C}$ is very hard to imagine. It's essentially a mapping from a 2 dimensional space to a 2 dimensional space, requiring 4 dimensions to show the mapping. In multivariable calculus, a function $f: \mathbb{R}^n \to \mathbb{R}^m$ is differentiable at a point if it has a **jacobian** matrix $\textbf{J}$ that is $m \times n$ and represents the approximate linear map at a particular point. The matrix is essentially an arbitrary linear mapping that's comparable to the tangent slope of a scalar real-valued function. Being holomorphic in $\mathbb{C}$ is similar in concept, except it has more restrictions. 

If we think about a scalar-valued function, we are essentially finding the *slope of the function*, which is a number in $\mathbb{R}$. Translating it to a complex-valued function, we are finding the same slope, but in $\mathbb{C}$, i.e. it's equivalent to multiplying with a complex number. What is multiplying with a complex number? It's equivalent to *a rotation and scaling action*. Wait... it's equivalent to the class of rotations and scaling in $\mathbb{R}^2$, and these are linear maps! **The restriction is thus the $$2\times 2$$ jacobian matrix for complex numbers is of the class of rotation matrices multiplied with scaling matrices.** In particular, it's of the form: 

$$
\begin{bmatrix}a & b \\-b & a\end{bmatrix} \in \mathbb{R^{2x2}}
$$

... in the interpretation of $\mathbb{C}$ as the [Argand plane](https://en.wikipedia.org/wiki/Complex_plane#Argand_diagram).

Because holomorphic functions are essentially a subset of real-valued multivariable differentiable functions, it carries with it the equivalent laws of differentiation. In the below statements, $f, g: \mathbb{C} \to \mathbb{C}$ and are complex differentiable at $z_0$.

1. **Linearity**: $h = f+g \implies h'(z_0) = f'(z_0) + g'(z_0)$ 

2. **Product rule**: $h = fg \implies h'(z_0) = f'(z_0)g(z_0) + f(z_0)g'(z_0)$

3. **Quotient rule**: $h = f/g \implies h'(z_0) = \frac{g(z_0)f'(z_0) - f(z_0)g'(z_0)}{g(z_0)^2}$
4. **Chain rule**: $h = f \circ g \implies h'(z_0) = f'(g(z_0))g'(z_0)$

One function that might be "smooth" but not holomorphic is $f(z) = \bar{z}$. Individually speaking, $f(Re(z)) = Re(z)$, and $f(Im(z)) = -Im(z)i$. Both of these seem relatively differentiable in their own right but it's not holomorphic:

$$
\text{lim}_{h\to 0} \frac{f(z_0+h) - f(z_o)}{h} = \frac{\bar{h}}{h}
$$

In the direction of $h$ from a real number, the value is 1. In the direction of $h$ from a complex number, the value is $-1$. The limit must be uniformly equal to $f'(z_0) \in \mathbb{C}$ for the function to be considered holomorphic. In the sense that the real component is $x$ and the imaginary component is $y$, if we look at the jacobian of $f(x, y) = (u(x, y), v(x,y)) = (x, -y)$ we also see that:

$$
\textbf{J} = \begin{bmatrix}\frac{\partial u(x,y)}{\partial x} & \frac{\partial u(x,y)}{\partial y} \\\frac{\partial v(x,y)}{\partial x} & \frac{\partial v(x,y)}{\partial y}\end{bmatrix}= \begin{bmatrix}1 & 0 \\0 & -1\end{bmatrix}
$$

the mapping does not conform to the generic type of rotation matrix with scaling. The **Cauchy-Riemann** equations essentially state this, with the following restrictions:

- $\frac{\partial u}{\partial x} = \frac{\partial v}{\partial y}$, which is the $a = a$ portion in the above matrix along the diagonal.
- $\frac{\partial u}{\partial y} = -\frac{\partial v}{\partial x}$, which is the $b = -(-b)$ portion in the above matrix across from the diagonal.

## Power Series

Some examples of power series are:

- The complex exponential: $e^z = \sum_{n=0}^\infty \frac{z^n}{n!}$ and the $sin(x)$ and $cos(x)$ functions that we've seen above.
- The geometric series: $\frac{1}{1-z} = \sum_{n=0}^\infty z^n$ for $\|z\| < 1$.

The general form of power series is in the form of:

$$
\sum_{n=0}^\infty a_nz^n \ \text{where }  a_n, z \in \mathbb{C}
$$

**Theorem: For any power series, there exists $R > 0$ such that if $\|z\| < R$ the series converges absolutely and $\|z\| > R$ the series diverges. This $R$ is called the radius of convergence, and is defined by $\frac{1}{R} = \text{limsup} \|a_n\|^{1/n}$.**

**Proof:** **Let's take care of edge cases.** Let $L = \frac{1}{R}$. If $L = 0 \implies R = \infty$, that means all convergent subsequences of form $\|a_n\|^{1/n}$ converges to 0. Then $a_n$ decays faster than $z^n$ grows, and thus the series is geometric and convergent.

If $L = \infty \implies R = 0$, that means there exists a single subsequence of form $\|a_n\|^{1/n}$ that diverges. A single divergent subsequence implies divergence of the series, and so we must set all terms to 0 to make sure the series converge.

**Let's take care of convergence.** We want to test that the series converges absolutely if $\|z\| < R$:

$$
\sum_{n=0}^\infty |a_nz^n| \leq \sum_{n=0}^\infty |a_n||z|^n < \infty
$$

Then we want to bound $\|a_n\|\|z^n\|$ by some geometrically decaying ratio. This ratio can be obtained in several ways, but fundamentally it's because of the two below statements:

- $\|z\| < R \implies \exists \delta > 0, \|z\|+\delta < R$  
- $\forall \epsilon > 0 \exists N \text{ such that } \forall n \geq N, \|\|a_n\|^{1/n} - \frac{1}{R}\| < \epsilon$ due to  $\text{limsup}$ of the sequence.

We start by picking some $\delta$ such that $\|z\| + \delta R < R$. For arbitrary $\epsilon$, there is a sufficiently large $N$ where for all $n \geq N, \|a_n\|^{1/n} - \frac{1}{R} < \frac{\epsilon}{R}$ (we removed the absolute value but it's still true). Then, we throw away all finite $k < N$ in the above series because finite series converge, and we observe:

$$
\sum_{n\geq N} |a_n||z|^n < \sum_{n\geq N} (\frac{1+\epsilon}{R})^n|z|^n < \sum_{n\geq N} (\frac{1+\epsilon}{R})^n((1-\delta)R)^n = \sum_{n\geq N} ((1+\epsilon)(1-\delta))^n
$$

We'll choose an $\epsilon$ sufficiently small so that the ratio $(1+\epsilon)(1-\delta) < 1$. Then the series converges geometrically.

**Let's take care of divergence.** We want to test that the series diverges absolutely if $\|z\| > R$. In that case, we'll modify the first statement to:

- $\|z\| > R \implies \exists \delta > 0, \|z\| > R + \delta$

In that case, we can pick some $\delta$ such that $\|z\| > R + \delta R$. For arbitrary $\epsilon$, there is sufficiently large $N, \forall n\geq N \|a_n\|^{1/n} > \frac{1}{R} - \frac{\epsilon}{R}$ (we removed the absolute value in the other way). We have the following inequality:

$$
\sum_{n\geq N} |a_n||z|^n > \sum_{n\geq N} (\frac{1-\epsilon}{R})^n|z|^n > \sum_{n\geq N} (\frac{1-\epsilon}{R})^n((1+\delta)R)^n = \sum_{n\geq N} ((1-\epsilon)(1+\delta))^n
$$

We'll choose an $\epsilon$ sufficiently small so that the ratio $(1-\epsilon)(1+\delta) > 1$. Then the series diverges. $\blacksquare$

---

On the disc of convergence boundary, convergence or divergence is not so easily determined, and is an interesting field of study by itself. For now, we can say some strong things about functions defined within the disc of convergence:

**Theorem: Within the disc of convergence, a power series function $f(z) = \sum_{n=0}^\infty a_nz^n$ is holomorphic with derivative $f'(z) = \sum_{n=0}^\infty n a_nz^{n-1}$.**

This proof is super tedious so... no. However, we can see that the derivative has the same radius of convergence as the original function. This is because $\text{limsup} \|a_n\|^{1/n} = \text{limsup} \|n a_n\|^{1/n}$ since the $\text{lim}_{n \to \infty} n^{1/n} = 1$ (applying sandwich theorem).

As a result of this finding, we see that **power series are infinitely complex differentiable!** Since power series are so important for complex analysis, we say that a function is **analytic** if it has a power series expansion, and thus also infinitely complex differentiable. 