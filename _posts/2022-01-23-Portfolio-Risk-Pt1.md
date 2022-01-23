---
published: true
title: Portfolio Risk - Part 1
use_math: true
category: math
layout: default
---

<style>
  .red {
    color:inherit;
  }
  .red:hover {
    color:rgb(129, 17, 18);
  }
  .collapse:hover {
    cursor: pointer;
  }
</style>

# Portfolio Risk - Part 1

The first step to making money in the stock market is having an accurate model of where the sources of profit(and losses) are coming from. We'll go through some basic (but time-tested) modelling assumptions and introduce some ways a portfolio manager might evaluate their positions in the market.

Disclaimer: I'm not responsible for any hillarious losses incurred by a portfolio inspired by methodologies listed below.

# Table of Contents

* TOC
{:toc}

## Returns and Log-returns

Making money is all about getting positive **returns**. One way to define return of holding an asset with price $p_t$ over 1 time-step is by measuring it in a comparative growth ratio:

$$
r_t = \frac{p_t}{p_{t-1}}
$$

This is great because returns became a unit-less number, one that doesn't change as the price of the asset becomes larger or smaller, which is great when we may potentially have many assets to worry about in a portfolio. Also, this equation is obviously a simplification of the real world, so it misses some details like dividends and stuff. With this, we can define **cumulative returns** as:

$$
R_t = \Pi_tr_t
$$

If we assume that $r_t$ is normally distributed which sort of makes sense if you were a [blindfolded monkey throwing darts at stocks](https://www.forbes.com/sites/rickferri/2012/12/20/any-monkey-can-beat-the-market/?sh=2e56b4fe630a), you would find that analyzing $R_t$ is going to be very hard because the product of gaussians is not gaussian. So as a solution to this, people tried to tractably model returns by assuming $p_t$ is distributed log normally, which kind of makes sense because prices must(at least in equities) be greater than zero. If we plug it in, we would see that $log(r_t)$ (log-returns) is actually normally distributed:

$$
log(r_t) = log(\frac{p_t}{p_{t-1}}) \sim \mathcal{N}(\mu, \sigma^2) \\
p_t, p_{t-1} \sim log\mathcal{N}(\mu, \sigma^2)
$$

This is great because we want to analyze cumulative log-returns, which is now:

$$
R_t = log(\Pi_tr_t) = \sum_t log(r_t) = log(p_t) - log(p_{t-1}) + log(p_{t-1}) - log(p_{t-2}) + ... - log(p_0) \\
= log(p_t) - log(p_0) \sim \mathcal{N}(\mu, \sigma^2)
$$

Which makes calculations tractable. There are also a couple other mathematical benefits to making this assumption, like accuracy of approximation on small returns. We won't dive into that rabbit hole in this blogpost, but you can find some good motivation [here](https://quantivity.wordpress.com/2011/02/21/why-log-returns/).

## Modeling Returns

Factor models are linear models that describe the return(not log-return) of a portfolio based off of a combination of factor exposures and idiosyncratic returns:

$$
r = Bf + \epsilon
$$

where $r \in \mathbb{R}^n, B \in \mathbb{R}^{n \times m}, f \in \mathbb{R}^m, \epsilon \in \mathbb{R}^n$. $r$ is the returns of a portfolio, $B$ is the factor loading matrix for each asset to each factor, $f$ is the factor returns, and $\epsilon$ is the idiosyncratic returns of the portfolio, which should(ideally) be a vector of uncorrelated random variables which capture the uniqueness of each asset in the portfolio. It is also worth noting that $\epsilon$ is uncorrelated from $f$ because it's residualized leftover from a regression using $f$, so it should have zero correlation with the factors as well. Here, idiosyncratic simply means returns not explained by the factor exposures. We can interpret this as a graphical, causal-inference model from underlying factors linearly impacting each asset's return with some weight. One important property we care about is the covariance matrix of returns, which can be expressed in terms of $\Sigma_f, \Sigma_\epsilon$:

$$
\Sigma_r = Var(r) = Var(Bf + \epsilon) = B\Sigma_fB^T + \Sigma_\epsilon
$$

Here, since $\epsilon$ can be considered uncorrelated(but potentially dependent!) random variables the matrix $\Sigma_\epsilon$ can be assumed diagonal. This is not always a good assumption though.

### When $\Sigma_\epsilon$ is not diagonal

If the idiosyncratic covariance is not diagonal, that means there is some residual structure amongst the assets that factors are not able to fully capture. If you have an ETF like SPY which contains TSLA as an underlier, and you also have TSLA in your portfolio, it's obvious that there are some idiosyncratic returns from TSLA that is correlated with SPY's returns. We would fix $\Sigma_\epsilon$ to be diagonal here by **securitizing** our holdings, which means we get rid of SPY as a real holding and turn it into its constituents(which isn't always an accurate as the net asset value doesn't always equal the book price of the ETF). Suppose we have a holdings vector $w \in \mathbb{R}^n$ in units of dollars, then securitization of $w$ to $w'$ is essentially a linear transformation:

$$
w' = Xw
$$

where $w' \in \mathbb{R}^{k}, X \in \mathbb{R}^{k \times n}, k > n$. You can imagine $X$ containing column vectors:

$$
X = \begin{bmatrix}
    \vert & \vert & ... & \vert \\
    x_1   & x_2   & ... & x_n \\
    \vert & \vert & ... & \vert
\end{bmatrix}
$$

where each element the column vector $x_i$ represents how many equivalent shares of the underliers (indexed in $k$ dimensions) the $i$-th original asset has. For example, if SPY was the $i$-th index asset in our portfolio, and TSLA is the $j$-th index asset in the securitized output, $X_{ij}$ would be 0.0144 (which is the current % asset breakdown by SPY holdings).

Let's think of another scenario, where the relationship between stocks are not so obvious. Suppose we're holding a few pharmaceutical companies that specialize in heart disease drugs. These companies don't own each other as an underlier, so securitization won't help us. It's obvious that they're correlated but a factor like "pharmaceutical heart-disease oriented company factor" seems extremely specific and should not be included in $f$  because the residual structure is miniscule in the universe of assets we worry about in a large portfolio. Then where can we describe this relationship? Another way we can visualize a non-diagonal $\Sigma_\epsilon$ is by taking a graphical approach, where the non-zero elements off the diagonal means inverse distance of an edge on a graph where each node is an asset. Using a graphical model here makes the risk model much more complicated (and potentially nonlinear), so we omit the details for this blog.

Here are some properties about the factor model, which we'll discuss first.

### Rank-preserving linear transformations

A factor model you get from Barra has semantic meaning for each factor exposure and loading vector, but you can apply a rank preserving linear transformation to the factors such that the returns are still the same. Let $C \in \mathbb{R}^{m \times m}$ be an invertible matrix, then:

$$
B' = BC \\
f' = C^{-1}f
$$

Then the original portfolio returns is still the same using these new transformed factor loadings/returns:

$$
r = B'f' + \epsilon = BCC^{-1}f + \epsilon = BC + \epsilon
$$

The covariance matrix is preserved under the transformation:

$$
Var(B'f' + \epsilon) = B'\Sigma_{f'}B'^T + \Sigma_\epsilon = (BC)(C^{-1}\Sigma_fC^{-T})(C^TB^T) + \Sigma_\epsilon = B\Sigma_fB^T + \Sigma_\epsilon
$$

If we want to have unit factor covariance and look at the transformed loadings, we can take $\Sigma_f = USU^T$ via SVD on symmetric matrices and take $C^{-1}= US^{\frac{-1}{2}}$ in the above example. This transformation normalizes the factors to be uncorrelated with each other.

We can also make the loading matrix $B$ orthonormal by taking the SVD of $B = USV^T$, and taking $C = VS^{-1}$.

### Projection

Projection is not a rank-preserving transformation but we still care about it. We want to minimize some norm of the difference of a lower rank approximation of $Bf$ which can be formalized as follows:

$$
min_g E(||Bf - Ag||^2)
$$

where $A \in \mathbb{R}^{n \times k}, g \in \mathbb{R}^k, k < m$. This is simple OLS which has a closed form solution if we are given the loadings $A$. The solution is:

$$
g = (A^TA)^{-1}A^TBf
$$

### Lifting

In contrast to projection, lifting is a transformation that increases the number of factors. This may be useful if $\epsilon$ exhibits some structure that can be reconsolidated as a factor:

$$
\epsilon = Ag + \epsilon' \implies r = Bf + Ag + \epsilon'
$$

To add the new factors $g$ and loadings $A$, we can just increase the length of the vector $f$ to include $g$ as the last few elements, and stack $B$ on top of $A$.

### Log-returns covariance transformation

If given our log-returns covariance matrix, then how do we know our actual returns' covariance matrix? Let the returns be $R \in \mathbb{R}^n, R_i = e^{r_i}, r = log(\frac{p_t}{p_{t-1}}) \sim \mathcal{N}(\mu_r, \Sigma_r)$. Here we want to calculate our actual return covariance matrix $\Sigma_R$ of the random vector $R$, which is returns (not logarithmic returns).

First, let's denote $\Sigma := \Sigma_r$, as it's unambiguous when defining $\Sigma_R$. For those uninterested in the derivation, the answer is:

$$
(\Sigma_R)_{ij} = exp(\frac{\Sigma_{ii} + \Sigma_{jj}}{2} + \mu_i + \mu_j) [exp(\Sigma_{ij}) - 1]
$$

<details><summary markdown='span' class='collapse'>**For those who are interested in the derivation**</summary>
We know that  $\Sigma_{ij}$ is the covariance of the i-th and j-th random variable of $R$. It can be formally expressed as:

$$
(\Sigma_R)_{ij} = cov(exp(r_i), exp(r_j)) = E(exp(r_i)exp(r_j)) - E(exp(r_i))E(exp(r_j))
$$

So there are two main quantities we want to find here. Let's tackle the easiest one first: $E(exp(r_i))$. For sake of simplicity, let's denote $x := r_i$ and $\mu := E(x) = \mu_i, \sigma^2 := Var(x) = \Sigma_{ii}$. 

$$
E(exp(x)) = \int_{-\infty}^{\infty} exp(x)p(x)dx = \frac{1}{\sqrt{2\pi}\sigma}\int_{-\infty}^{\infty}exp(x)exp(-\frac{(x-\mu)^2}{2\sigma^2})dx
$$

As we can see here, $e^x$ can be thought of as performing a shift and a scaling to the original pdf of the gaussian:

$$
\frac{1}{\sqrt{2\pi}\sigma}\int_{-\infty}^{\infty}exp(\frac{2\sigma^2x - (x-\mu)^2}{2\sigma^2})dx
$$

We can expand the quantity and complete the square on top to get:

$$
2\sigma^2x - (x-\mu)^2 = -[x^2 - 2x(\mu + \sigma^2) + \mu^2] =  -(x-(\sigma^2+\mu))^2 + \sigma^4 + 2\mu\sigma^2
$$

Using this quantity on the numerator, we can refactor the equation to look like:

$$
\frac{1}{\sqrt{2\pi}\sigma}\int_{-\infty}^{\infty}exp(\frac{2\sigma^2x - (x-\mu)^2}{2\sigma^2})dx = \frac{exp(\mu + \frac{\sigma^2}{2})}{\sqrt{2\pi}\sigma}\int_{-\infty}^{\infty}exp(\frac{-(x - (\sigma^2+\mu))^2}{2\sigma^2}) = exp(\mu + \frac{\sigma^2}{2})
$$

So looking back at what we substituted, we get the quantity:

$$
E(exp(r_i))E(exp(r_j)) = exp(\mu_i + \mu_j + \frac{\Sigma_{ii} + \Sigma_{jj}}{2})
$$

Now let's look at the slightly harder quantity $E(exp(r_i)exp(r_j))$. We can solve the expectation in a similar way, but this time with matrix version of completion of the square. I didn't want to reprove completion of the square in matrix form so I just looked up the equation [here](https://gregorygundersen.com/blog/2019/09/18/completing-the-square/):

$$
x^TMx - 2b^Tx = (x-M^{-1}b)^TM(x-M^{-1}b) - b^TM^{-1}b \tag{1}
$$

The general pdf for a multivariate gaussian random variable $r \sim \mathcal{N}(\mu, \Sigma)$  is:

$$
\int_{-\infty}^{\infty}f(r) = \int_{-\infty}^{\infty} \frac{1}{\sqrt{(2\pi)^n|\Sigma|}}exp(\frac{-1}{2} (r-\mu)^T\Sigma^{-1}(r-\mu))
$$

The expectation we're trying to solve for can be re-written as:

$$
E(exp(r_i)exp(r_j)) = E(exp(r_i+r_j))
$$

Here, $r_i$ and $r_j$ are scalar random variables, so in order to incorporate it into equation $(1)$ we make the quantity $r_i + r_j = e_{ij}^Tr$ where $e_{ij}^T = (0 \text{...} 1 \text{...} 1 \text{...} 0)$ - a vector with 1's on the i-th and j-th indices. We get this:

$$
\int_{-\infty}^{\infty} \frac{1}{\sqrt{(2\pi)^n|\Sigma|}}exp(\frac{-[(r-\mu)^T\Sigma^{-1}(r-\mu) - 2e^T_{ij}(r-\mu) - 2e^T_{ij}\mu]}{2})
$$

Note that we did some extra work to express the last term in the form of $r-\mu$ .We get the following completion setting $r-\mu = x,\Sigma = M,b = e^T_{ij}$:

$$
((r-\mu)-\Sigma e_{ij})^T\Sigma^{-1}((r-\mu) - \Sigma e_{ij}) - e_{ij}^T\Sigma e_{ij} \\
= (r-(\mu+\Sigma e_{ij}))^T\Sigma^{-1}(r-(\mu+\Sigma e_{ij})) - e_{ij}^T\Sigma e_{ij}
$$

The completed square with the remaining $-2e^T_{ij}\mu$ term can be tucked back into the pdf to be integrated to 1 with $\mu' := \mu + \Sigma e_{ij}$, so we get:

$$
\int_{-\infty}^{\infty} \frac{1}{\sqrt{(2\pi)^n|\Sigma|}}exp(\frac{-1}{2} [(r-\mu')^T\Sigma^{-1}(r-\mu') - e_{ij}^T\Sigma e_{ij} - 2e^T_{ij}\mu]) \\
= exp(\frac{e^T_{ij}\Sigma e_{ij} + 2e^T_{ij}\mu}{2})
$$

We're almost there. The quadratic form of the above expression can be evaluated to:

$$
tr(e^T_{ij}\Sigma e_{ij}) = tr(e_{ij}e^T_{ij}\Sigma) = \Sigma_{ii} + \Sigma_{ij} + \Sigma_{ji} + \Sigma_{jj} = \Sigma_{ii} + 2\Sigma_{ij} + \Sigma_{jj}
$$

We can cyclically permute traces, so we rearrange to get the outer product of $e_{ij}e^T_{ij}$ with $\Sigma$. This essentially sums up all elements in that block, and the last equality is due to symmetry. Now the entire expression is:

$$
E(exp(r_i)exp(r_j)) - E(exp(r_i))E(exp(r_j)) \\
= [exp(\frac{\Sigma_{ii} + 2\Sigma_{ij} + \Sigma_{jj} + 2e^T_{ij}\mu}{2})] - [exp(\mu_i + \mu_j + \frac{\Sigma_{ii} + \Sigma_{jj}}{2})] \\
= [exp(\frac{\Sigma_{ii} + \Sigma_{jj}}{2} + \mu_i + \mu_j + \Sigma_{ij} )] - [exp(\mu_i + \mu_j + \frac{\Sigma_{ii} + \Sigma_{jj}}{2})] \\
= exp(\frac{\Sigma_{ii} + \Sigma_{jj}}{2} + \mu_i + \mu_j) [exp(\Sigma_{ij}) - 1]
$$

</details>
{: .red}

<details><summary markdown='span' class='collapse'>**Here's some sanity checking for the cynical**</summary>
We first generate samples of $r$ and create a randomly generated correlation matrix of $r$. We first calculate the matrix $\Sigma_R$ from an estimated $\Sigma_r$ as per equation above, and we compare that to covariance matrix calculated from directly exponentiating each sample of $r$.


```python
from scipy.stats import random_correlation
import numpy as np
import seaborn as sb
import matplotlib.pyplot as plt

num_samples = 100000
dimensions = 10
scale = 0.1
rng = np.random.default_rng()
random_eigs = np.array([np.random.uniform() for _ in range(dimensions)])

# Normalize the random eigs so it all sums to N
random_eigs = random_eigs / random_eigs.sum() * dimensions
cov = random_correlation.rvs(random_eigs, random_state=rng)
means = np.random.normal(scale=scale, size=dimensions)
samples = np.random.multivariate_normal(means, cov, size=num_samples)
exp_samples = np.exp(samples)
estimated_cov = np.cov(samples, rowvar=False)
estimated_mean = np.mean(samples, axis=0)


def from_log(samples):
    estimated_cov = np.cov(samples, rowvar=False)
    estimated_mean = np.mean(samples, axis=0)
    cov = np.zeros_like(estimated_cov)
    for i in range(cov.shape[0]):
        for j in range(cov.shape[1]):
            s_ii, s_jj, s_ij = estimated_cov[i, i], estimated_cov[j, j], estimated_cov[i, j]
            m_i, m_j = estimated_mean[i], estimated_mean[j]
            cov[i, j] = np.exp((s_ii + s_jj) / 2 + m_i + m_j) * (np.exp(s_ij) - 1)
    return cov

# NOTE: These are two separate samples from the same distribution, so slight differences
# are expected. You can also check their frobenius norm via np.linalg.norm(x-y)
sb.heatmap(from_log(samples), cmap="Blues")
sb.heatmap(np.cov(exp_samples, rowvar=False), cmap="Blues")
```
Here's the two graphs (which should look identical):

**Calculated covariance from $R$**:

![portfolio_calculated_covariance.png]({{ site.url }}/assets/portfolio_calculated_covariance.png)

**Derived covariance from $\Sigma_r$**:

![portfolio_derived_covariance.png]({{ site.url }}/assets/portfolio_derived_covariance.png)

</details>
{: .red}

## Portfolio Optimization

With a certain risk threshold in mind, we can optimize our portfolio to maximize returns. One way of formalizing this is to maximize $w$:

$$
\text{max}_w \alpha^Tw \\
\text{s.t.} w^T\Sigma_r w \leq \sigma^2
$$

Where $\alpha := E(r), \Sigma_r := Var(r)$.

Formulated with lagrange multipliers, we get a loss function:

$$
V(w) = \alpha^Tw - \frac{\lambda}{2}w^T\Sigma_r w
$$

You can think of $V(w)$ as the "value" of your portfolio given the expected returns and covariance. This is a concave function with a single extrema (the maxima, which we're trying to find), so we can set the gradient to zero and solve for $w$:

$$
\nabla_w V(w^*) = \alpha - \lambda \Sigma_r w^* = 0 \\
\implies w^* = \frac{1}{\lambda}\Sigma_r^{-1}\alpha
$$

If we didn't care about the covariance term, $\alpha$ would be an arbitrarily large vector in the span of $\alpha$. However, because we are aware that some assets are more volatile than others and we only have so much room for risk in our portfolio, we "rotate" our assets so that it maximizes our total returns on a particular gradient contour(in this case, shaped like an N-dimensional ellipse).

Unfortunately, this is actually *not* a great equation to optimize on because $\alpha$ and $\Sigma$ are estimates of the true expected returns and covariance matrices. I haven't found a rigorous proof for this (comment below if you have any sources!), but my intuition is that this particular solution of $w^*$ is particularly sensitive to the change in the covariance matrix and expected returns (I'm guessing moreso the covariance matrix as it's being inverted) which makes the solution hard to generalize.

### Robust-er formulation

Instead of taking the estimated $\alpha$ and $\Sigma$ as constants, we can model $\alpha \sim \mathcal{N}(\mu_\alpha, \Sigma_\alpha)$, and introduce an additional gaussian noise term $\epsilon$ to the optimization:

$$
V(w) = \mu_\alpha^Tw - \frac{\lambda}{2}w^T(\Sigma_\alpha+\Sigma_r) w
$$

Maximizing this gives us:

$$
w^* = \frac{1}{\lambda}(\Sigma_\alpha + \Sigma_r)^{-1}\mu_\alpha
$$

A (computationally) efficient assumption of $\alpha$'s distribution is that these random variables are uncorrelated, which means $\Sigma_\alpha$ is actually a diagonal matrix. If we assume the assets have similar volatility, this solution starts to look like [ridge regression](https://en.wikipedia.org/wiki/Ridge_regression).

---

Note that the solutions to the above formulations only take care of maximizing $w$ for a single time interval, and the real problem lies in optimizing:

$$
\text{max}_{w_1, w_2, ..., w_T} \Sigma_t^T V(w_t)
$$

which is a much harder trajectory problem, requiring heavier hammers like the [linear-quadratic regulator](https://en.wikipedia.org/wiki/Linear%E2%80%93quadratic_regulator), which is commonly used in control theory and reinforcement learning.

## Performance Attribution

When we look at PnL and attribution to factor returns or idiosyncratic return, we can decompose it as follows for a given time:

$$
\text{pnl} = \sum_t\text{pnl}_t \\
= \sum_t w_t^Tr_t \\
= \sum_t w_t(B_tf_t) + \sum_tw_t^T\epsilon_t := \sum_t b_t^Tf_t + \sum_tw_t^T\epsilon_t \\
\approx \text{factor pnl}+\text{idiosyncratic pnl}
$$

Where $b_t \in \mathbb{R}^m$ and is the factor exposure of the portfolio. **For a systematic investor, we care about factor pnl, and for fundamental investor, we care about the idiosyncratic pnl**. This risk attribution equation seems good, but it's a trap - $f_t$ and $e_t$ are estimated! If we suppose $f'_t, e'_t$ are the true factor returns and idiosyncratic return then we have accrued $\sum_t w_t^T(f_t - f'_t)$ error in our factor estimates and similarly for the idiosyncratic term. If we assume our position $w$ doesn't change for simplicity and $f_t - f'_t \sim \mathcal{N}(\mu, \sigma^2)$ and are independent in time, summing gaussians will give us linearly increasing variance which is catastrophic to our error bounds if we were estimating cumulative returns with long intervals. This is to say much like the portfolio optimization problem, performance attribution becomes a harder problem when we're dealing across a long time horizon.

## Stress Tests

You've probably heard of 3-$\sigma$ events. What does that exactly mean? First, we need to define **volatility** of a portfolio:

$$
\sigma = \sqrt{w^T\Sigma_rw}
$$

which is the standard deviation of returns. If we assume that returns are independent over time T, we get:

$$
\sigma_T = \sqrt{\sum_i^Tw_i^T\Sigma_{r_i}w_i}
$$

If the volatility is fairly constant (which is usually the case when we're looking at sub-month intervals), some people just abbreviate this to:

$$
\sigma_T \approx \sigma\sqrt{T}
$$

A 3-$\sigma$ event is when your observed event (returns) is 3 times the volatility away from the mean. This could mean we made a ton of money or lost a ton of money. You may run into 3-$\sigma$ events more often than expected, and this isn't only due to the imperfect art of modelling returns. By definition, the $\Sigma_r$ matrix is fixed in the calculation, but this isn't true in practice. In a catastrophic event (i.e. Elon Musk tweets the world is going to end in 5 days), the correlation across assets suddenly approaches 1 and the condition number of the covariance matrix skyrockets (the higher the condition number, the more sensitive the estimated returns are to small perturbations in $\Sigma_r$ or $w$). A better way to model an extreme event is to calculate the worst-case transformation of $\Sigma_r$ under stress:

$$
\max_M \sqrt{w^TMw} \\
\text{s.t.} ||\Sigma_r - M||_F < C
$$

Which is a better model for extreme events. Lots of stress evaluation metrics such as [value-at-risk](https://en.wikipedia.org/wiki/Value_at_risk) can use the covariance matrix to calculate maximum loss.

## Conclusion

Portfolio modellings foundations rest on handcrafted factors, domain knowledge, and lots of linear regression. We first defined what a return is, and move on to modelling the returns using factor models. Then, we mentioned a few ways to formulate the modelling as an optimization problem so we can find the best positions for our portfolio. Lastly, we look at how to attribute money gained/lost and the worst case scenarios that may occur.  There's a lot of stuff I didn't yet mention (i.e. performance attribution) which I might include in a subsequent post, but I hope this survey into the different parts of risk/portfolio modelling helps!

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
