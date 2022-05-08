---
published: false
title: Portfolio Risk - Part 2
use_math: true
category: math
layout: default
---

# Portfolio Risk - Part 2: Evaluation

This is a continuation of the [previous post](https://oneraynyday.github.io/math/2022/01/23/Portfolio-Risk-Pt1/), so I'll be skipping some preliminary setup. 

Noone makes a good risk model in one go. It's important to understand how to evaluate a risk model in order to iteratively improve on it with respect to those performance characteristics. Unlike a typical supervised machine learning problem, evaluating a risk model is a much more involved task.

## Qualitative evaluation

Before we dive into the quantative aspects of a risk model, there are some qualitative checks we should do to ensure the model is well defined and adequately captures the structure of asset returns.

### Loadings pervasiveness & degeneracy

A factor model's loading $B$ can be either observable metrics of a given asset or a result of a statistical model. For an example, if we take the market cap of an asset as its loadings, we would essentially be describing the asset's exposure to the _size_ factor. These $m$ loadings would be column vectors in $B$:

$$
B = \begin{bmatrix}
    \vert & \vert & ... & \vert \\
    b_1   & b_2   & ... & b_f \\
    \vert & \vert & ... & \vert
\end{bmatrix}
$$

Loadings for a large factor model should be pervasive. This roughly means $||b_i|| = O(n)$ i.e. the norm of each loading should scale roughly linearly to the number of $n$ assets. If there are many zeros in a particular loading, it may be trying to capture small group structure amongst the assets and is not a good candidate as a factor.

Sometimes loadings are linearly dependent (or close enough) which leads to degeneracy during regression. Using ordinary least squares, we need to invert the gram matrix of $B$. If there is no ridge term, we would get an error during estimation.

$$
(B^TB)^{-1}B^Tr = f
$$

The offending factors can be identified using Gram Schmidt and removed during the estimation phase. The US barra model for example has the linearly dependent factor "country" which is a linear combination of all the "industry" factors that makes up the country's asset universe.
 
### Idiosyncratic covariance

The idiosyncratic covariance matrix here is an **ex post**(measured from historical data) statistic:
$$
\Omega_i := \Omega_r - B\Omega_fB^T
$$
where $B \in \mathbb{R}^{m \times f}, \Omega_r \in \mathbb{R}^{m \times m}, \Omega_f \in \mathbb{R}^{f \times f}$. The loadings $B$ is often constructed statistically (e.g. using PCA or other dimensionality reduction techniques) or fundamentally (e.g. using asset characteristics like market cap). The returns and factor covariance matrices can be estimated in a few ways:

1. Using exponential weighted moving average updates of centered outer products of historical asset/factor returns to form a full rank covariance matrix.
2. Fit a multivariate [GARCH](https://en.wikipedia.org/wiki/Autoregressive_conditional_heteroskedasticity) model over the observed asset/factor returns to generate per-period covariance matrices.
3. Estimate a [Wishart distribution](https://en.wikipedia.org/wiki/Wishart_distribution) from historical asset/factor returns and use the parameter $\Sigma$  as a covariance estimate.

Regardless of the estimation technique, we would like to get $\Omega_i$ and visualize the highly correlated clusters.

#### Correlation clustering

The problem of correlation clustering is NP complete, and can be framed as an graph optimization problem. Given a undirected graph $G(V, E)$, the output of correlation clustering is $C = \{C_1, C_2, ..., C_k\}$ clusters where $C_i = (V_i, E_i)$ such that $\{V_i\}$ is a partition of the vertices, and edges are the corresponding edges that belong to all nodes in each cluster. In our case, the graph node are the $n$ assets and the $n^2$ edges are the correlations between assets (sometimes we use the absolute value of correlation here. For simplicity we just take the raw correlation below). The more negative the correlation of assets $i$ and $j$, the less we'd like to include assets $i$ and $j$ in the same correlation cluster. Rigorously, define a $\delta$ function:

$$
\delta(i,j) = \begin{cases}
1 \quad \text{if i, j are in same cluster} \\
0 \quad \text{if i, j are in different clusters}
\end{cases}
$$

then:

$$
f_+(C) = \sum_{i,j: e_{i,j} > 0} -e_{i,j} \delta(i,j) + \sum_{i,j: e_{i,j} > 0} e_{i,j} (1-\delta(i,j))
$$

where the left term is the reward for putting correlated nodes in the same cluster, and the right term is the penalty for putting correlated nodes in different clusters. Similarly for anti-correlated nodes:

$$
f_-(C) = \sum_{i,j: e_{i,j} < 0} e_{i,j} \delta(i,j) + \sum_{i,j: e_{i,j} < 0} -e_{i,j} (1-\delta(i,j))
$$

where the left term is the penalty for putting anticorrelated nodes in the same cluster, and the right term is the reward for putting anticorrelated nodes in different clusters. We would like to minimize the following:

$$
f(C) := f_+(C) + f_-(C)
$$

and the correct clustering is:

$$
C* = argmin_C f(C)
$$

Unfortunately we cannot phrase this in a linear programming setup since membership of clusters can only be decayed to an integer linear programming problem (which is NP complete). Luckily, we have a nondeterministic 3-approximation algorithm producing $\hat{C}$, where 3-approximation is defined as:

$$
f(\hat{C}) \leq 3 f(C*)
$$

The procedure looks like (actual code available in a [gist](https://gist.github.com/OneRaynyDay/8a7c5a2342533f7262e1e9f423aa9971)):

```python
def cluster(vertices):
	v := random vertex
	c := all nodes with positive edges to v (including v)
	[c1, ..., cn] := cluster(vertices - c)
	return [c, c1, ..., cn]
```

For illustration purposes, here's a synthetic covariance matrix of 10 assets:

![corr]({{ site.url }}/assets/clustering/original_corr.png)

This is after we re-ordered the row/column axes according to clusters:

![reordered_corr]({{ site.url }}/assets/clustering/clustered.png)

As we can see, the clustering did a pretty good job at including positive edges(darker colors is better):

![cluster_mask]({{ site.url }}/assets/clustering/cluster_mask.png)

... and excluding negative edges:

![without_cluster_mask]({{ site.url }}/assets/clustering/without_cluster_mask.png)

Of course, there are also other approaches to clustering that may be useful, such as hierarchical clustering using dendrograms (example visualization [here](https://stackoverflow.com/a/3017704/3781180)). It's always a good idea to inspect the idiosyncratic correlation matrix clusters to get an intuitive idea of the relationship between assets in your investment universe.


#### Microstructure

With $f << m$, there must be some local structure the factor model cannot accurately capture. For example, there are only a handful of major public bank stocks which have very highly correlated historical returns. Another example is the `INTL` vs. `AMD` marketshare battle - when one company gets a major data center contract for a million server-grade CPU's, the other company loses that contract. These two stoks exhibit consistently negative correlation in less volatile market conditions. In both cases, adding a new banking factor or a microprocessor manufacturing factor for a small set of stocks would violate the pervasiveness requirement of loadings.

The astute reader may realize that there is no solution to this qualitative problem. This is inherently a problem with factor models, and the modeler should use the right tool for the job. If the investment universe only consists of tightly correlated assets with significant microstructure, maybe a factor model isn't the right approach.

#### Macrostructure

Sometimes, a factor model _is_ the right approach but we're lacking some important factors that influence the returns of a large set of assets. This usually means there are macro correlation clusters that should be encapsulated by factors. Quantitatively, this can be found by performing PCA on the idiosyncratic covariance matrix and getting the first few largest eigenvalues. If our factor model is large & optimal, we would assume the eigenvalues to be very small. If the eigenvalues are large we can incorporate the eigenvectors as loadings or interpret the eigenvectors to be some properties about the assets(which makes the factors more tradeable).

However depending on the purpose of the risk model, this isn't always a relevant performance characteristic to consider. For example, a risk model with very few but tradeable factors (e.g. index tracking factors) that explain the majority of the variance of a portfolio is a great *hedging* tool. Having very few tradeable factors means it is feasible to hedge a strategy based on the model (it would be really hard to hedge a portfolio using 100+ instruments). On the other hand, the idiosyncratic covariance matrix is bound to have lots of uncaptured macrostructure. This is often a tradeoff a portfolio manager is willing to make.

##### Marchenko-Pastur

Similar to asking "is there macrostructure in our idiosyncratic covariance matrix", we can ask "does the idiosyncratic covariance matrix match our assumptions?". We estimated our factor model such that the **idiosyncratic returns are zero-centered gaussian noise**. What's the likelihood that our assumptions are correct?

In the field of random matrix theory (RMT), we'll loosely[^1] define a random matrix drawn from a Wishart ensemble as a matrix $X$ constructed from the following steps:

First create a matrix $M \in \mathbb{R}^{n \times m}$ such that $M_{i,j} \sim \mathcal{N}(\mu,\sigma^2)$ are independently sampled. This matches our assumptions if for each row in $M$ we have the idiosyncratic returns for that day.

We then symmetrize the matrix by taking $X := \frac{M^TM}{n}$. Assuming idiosyncratic returns have mean 0, this is the covariance matrix. The eigenvalues of a symmetric matrix are real and non-negative, so it's a positive semidefinite matrix. The joint probability distribution of this matrix is proportional to:

$$
p(X) \propto e^{\frac{-1}{2}tr(X) det(X)^{\frac{1}{2} (n-m-1)}}
$$

Two random dudes named Marchenko and Pastur were looking at the spectral density of the Wishart ensemble by asking the question "what happens to the distribution of eigenvalues of $X$ asymptotically as $n$ and $m$ increase?". As $m$ increases, $X$ becomes a larger square matrix with more non-negative eigenvalues. As $n$ increases, we can use the law of large numbers to reach some convergence guarrantees. Under the assumption that $n$ and $m$ grow in some asymptotic ratio $c := m/n$, they were able to find the spectral density function $f(x)$ as:

$$
f(x) = \frac{1}{2\pi c x}\sqrt{(b-x)(x-a)}
$$

with support $[a,b]$, where $a,b = \pm(1-\sqrt{c})^2$. The distribution looks like this with $c < 1$:

![marchenko1]({{ site.url }}/assets/marchenko1.png)

For $c > 1$:

![marchenko2]({{ site.url }}/assets/marchenko2.png)

If we can derive a reasonable $c$ either by getting a maximum likelihood estimate from our idiosyncratic covariance matrix or by an educated forecast, we can calculate the likelihood that the eigenvalues of our idiosyncratic covariance matrix is sampled from the Marchenko-Pastur distribution.

##### John's sphericity test

Marchenko and Pastur have given us a useful tool to measure how likely our idiosyncratic covariance matrix belongs to a Wishart ensemble (the underlying idiosyncratic returns is gaussian noise). Another trait we've assumed about our idiosyncratic covariance matrix is that **the off-diagonal entries should be zero** as in the noise is uncorrelated. Some random dude named John(last name) came up with this statistic in the 1970's:

$$
U = \frac{1}{m}tr((\frac{C}{tr(C)/m)} - I)^2)
$$

Here, $C$ is our **correlation** matrix(which can be directly derived from $\Omega$, our covariance matrix). Since trace is rotation invariant, the quantity inside is roughly equivalent to the squared error between the eigenvalues of a spherical distribution and the elliptical idiosyncratic covariance distribution. The score itself is hard to comprehend, and is often used to compare two different models under the same symbol universe (for same $m$) - a lower score is "more diagonal" and better.

## Loss functions

Loss functions is a straightforward method to quantitatively reason about the performance of a model. Loss functions output a number - if it's high we're sad and if it's low we're happy. The number usually isn't interpretable so it is only useful when there are multiple numbers corresponding to different models to compare with. 

Typical supervised learning problem often have simple loss functions. A common choice is to use the negative log-likelihood as your loss function with some regularization terms in the lagrangian. The intuition is simple - try to select an estimator from a family of function with relatively low bias and variance.

In the context of backtesting factor models, we can use loss functions to evaluate the accuracy of predicted portfolio variance. Concretely, variance loss functions are mappings of the form:

$$
f: \mathbb{R}_+^T \times \mathbb{R}_+^T \to \mathbb{R}
$$

This reads "the function takes in two vectors containing positive values indexed by time and outputs a real number". One of the positive vector arguments is the actual portfolio variance across days, and the other is the predicted portfolio variance across days. We can *almost* consider the actual portfolio variance as the labels of the training set. Unfortunately, we can't observe the variance of any assets at a single point in time! Asset variance is a nonstationary, latent quantity so we don't have any labels for our predictions! What do we do?

### Proxies (ex post)
Proxies are able to give a reasonable volatility estimate to use as labels to compare against predictions. A simple volatility proxy can be computed as:

$$
\sigma^2 = (w^Tr)^2

\sigma = |w^Tr|
$$

The above proxy is _very_ noisy but has less bias than some types of estimates, making this a bias-variance tradeoff problem. We now have some noisy estimates of our labels, which isn't the most satisfying but we tread onwards.

### Predictors (ex ante)
Given a factor model, we can calculate the covariance matrix using:

$$
\hat{\Omega} = B\Omega_fB^T + \Omega_i
$$

where $\Omega_i$ is a diagonal covariance matrix, $B$ is the loadings of symbols to factors, and $\Omega_f$ is the full factor covariance matrix.

Given a portfolio $w \in \mathbb{R}^m$, we can predict its variance:

$$
\hat{\sigma}^2 = w^T \hat{\Omega} w
$$

Now that we have the two inputs to a loss function, let's choose our functions!

### MSE

### QLike

## Test portfolios

### Historical strategy portfolios

### Benchmark portfolios

### Minimum variance portfolios

### Hedged portfolios

## Statistics

### Turnover

### t-statistic on factor returns

### Singular values of correlation matrix



[^1]: I lied to make this simpler. For a general Wishart ensemble the pdf is parametrized with $\beta$ in the exponentiation and is called the dyson index. For the construction we defined, $\beta = 1$ because we've constructed a wishart ensemble using real numbers in $M$.