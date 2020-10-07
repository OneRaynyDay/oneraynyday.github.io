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

- The **norm form** of a complex number $z$ is a measure of its distance from the origin: $$N(z) = a^2 + b^2$$, and is the squared distance: $$N(z) = \|z\|^2$$. The **bilinear form** of two complex numbers $$z,w$$ is defined as $$\langle z, w \rangle = Re(z\bar{w}) = \frac{(ac-b(-d))^2+(a(-d) + bc)^2}{2} = \frac{(ac+bd)^2 + (-ad + bc)^2}{2}$$.

- The **$$Re : \mathbb{C} \to \mathbb{R}$$** gives the real part of a complex number $z$ is $a$, and can be derived as $$Re(z) = \frac{z+\bar{z}}{2}$$. Similarly, the **$$Im : \mathbb{C} \to \mathbb{R}$$** part of a complex number is $$b$$ and can be derived as $$Im(z) = \frac{z - \bar{z}}{2i}$$. We have the property $$\|Re(z)\|, \|Im(z)\| \leq \|z\| \leq \|Re(z)\| + \|Im(z)\|$$ (draw a triangle).

- The **addition** of two complex numbers $z, w$ is defined as $z+w = (a+c) + (b+d)i$. This is a translation mapping in the real and imaginary axes.

- The **subtraction** of two complex numbers $z,w$ is defined as $z-w = (a-c) + (b-d)i$.

- The **multiplication** of two complex numbers $$z,w$$ is defined as $$zw=(a + bi)(c+di) = ac + adi + cbi + bdi^2 = (ac - bd) + (ad + bc)i$$. This is a rotation and scaling operation if we interpret complex numbers as polar coordinates.

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

If we look at $e^{-ix} = cos(-x) + isin(-x) = cos(x) - isin(x)$, we can derive that $sin(x) = \frac{e^{ix} - e^{-ix}}{2i}$ and $$cos(x) = \frac{e^{ix} + e^{-ix}}{2}$$, which look very familiar to the [hyperbolic trigonometric functions](https://en.wikipedia.org/wiki/Hyperbolic_functions), which are the $sin, cos$ variants rotated-by-90 degrees in the clockwise direction in the complex plane.

# Complex differentiation

Given a function $f: \mathbb{C} \to \mathbb{C}$, it is **differentiable** at $z_0$ if:

$$
lim_{h \to 0} \frac{f(z_0 + h) - f(z_0)}{h} = f'(z_0) \in \mathbb{C}
$$

Or, by delta epsilon definitions, it is differentiable at $z_0$ with derivative $f'(z_0)$ if $\forall \epsilon > 0 \exists \delta > 0$ such that for all $\|z-z_0\| < \delta$:

$$
|\frac{f(z)-f(z_0)}{z-z_0} - f'(z_0)| < \epsilon
$$

If $f$ is differentiable everywhere in its domain, then it's called **holomorphic** or **complex differentiable**. The reason we don't call it differentiable simply is because holomorphicity is _stronger_ of a property. As opposed to a real-valued function $f: \mathbb{R} \to \mathbb{R}$ which is easy to draw and visualize, $f: \mathbb{C} \to \mathbb{C}$ is very hard to imagine. It's essentially a mapping from a 2 dimensional space to a 2 dimensional space, requiring 4 dimensions to show the mapping. In multivariable calculus, a function $f: \mathbb{R}^n \to \mathbb{R}^m$ is differentiable at a point if it has a **jacobian** matrix $\textbf{J}$ that is $m \times n$ and represents the approximate linear map at a particular point. The matrix is essentially an arbitrary linear mapping that's comparable to the tangent slope of a scalar real-valued function. Being holomorphic in $\mathbb{C}$ is similar in concept, except it has more restrictions. 

If we think about a scalar-valued function, we are essentially finding the *slope of the function*, which is a number in $\mathbb{R}$. Translating it to a complex-valued function, we are finding the same slope, but in $\mathbb{C}$, i.e. it's equivalent to multiplying with a complex number. What is multiplying with a complex number? It's equivalent to *a rotation and scaling action*. Wait... it's equivalent to the class of rotations and scaling in $\mathbb{R}^2$, and these are linear maps! **The restriction is thus the $$2\times 2$$ jacobian matrix for complex numbers is of the class of rotation matrices multiplied with scaling matrices.** In particular, it's of the form: 

$$
\begin{bmatrix}
a & b \\
-b & a
\end{bmatrix} \in \mathbb{R^{2x2}}
$$

... in the interpretation of $\mathbb{C}$ as the [Argand plane](https://en.wikipedia.org/wiki/Complex_plane#Argand_diagram).

Because holomorphic functions are essentially a subset of real-valued multivariable differentiable functions, it carries with it the equivalent laws of differentiation. In the below statements, $f, g: \mathbb{C} \to \mathbb{C}$ and are complex differentiable at $z_0$.

1. **Linearity**: $h = f+g \implies h'(z_0) = f'(z_0) + g'(z_0)$ 

2. **Product rule**: $h = fg \implies h'(z_0) = f'(z_0)g(z_0) + f(z_0)g'(z_0)$

3. **Quotient rule**: $h = f/g \implies h'(z_0) = \frac{g(z_0)f'(z_0) - f(z_0)g'(z_0)}{g(z_0)^2}$
4. **Chain rule**: $h = f \circ g \implies h'(z_0) = f'(g(z_0))g'(z_0)$

