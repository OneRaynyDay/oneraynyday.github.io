---
published: true
title: Portfolio Risk ELI5
use_math: true
category: math
layout: default
---

It's been a while since I posted! This ELI5 article is meant for those who are curious about portfolio management but don't know anything about portfolio construction. Here, I try to cover a breadth of topics rather than going very deep in one. I also explicitly oversimplify some concepts here to make it easier to digest, and we'll try to dispel some of these lies in a later blog.

# Table of Contents

* TOC
{:toc}


## Problem Statement - Make some $$$

What does it mean to "win" at the stock market? Do you want to become a bitcoin millionaire and retire early, or do you want to gradually size up a diverse portfolio that gives you performance marginally better than the S&P500 with relatively little risk, and ride the exponential growth until you retire?

To a greater extent, portfolio management tries to solve the latter problem. Most portfolio managers acrue large positions with leverage and try to beat the broader market in returns in order to pay themselves and grow their clients' net worth. How do they do this?

## Modelling for Returns

You might have heard of the terms "alpha" and "beta" used in a financial context before. They're terms used in a family of models that portfolio managers use, and are the main subjects of research in a typical hedge fund.

### The avocado alpha

Trading firms have access to a vast amount of data which allows them to identify specific high-performing **alphas** (denoted $\alpha$) which are also called *expected returns*. You can think of high-performing alphas as a piece of predictive knowledge that allows someone to construct a portfolio with greater returns than the market index. Imagine you discovered an imminent earthquake is about to hit California's central valley and destroy all the avocado farms. The price of avocados would spike shortly after, but if you beat everyone to the punch, you would buy a ton of avocados[^1] and sell them when the supply is low(and subsequently when the price is high).

If you discovered an avocado alpha, you wouldn't want to tell others until you've bought enough avocados to make a sizeable profit. This is why trading firms are so secretive with how they determine their investments.

### Two tales of zero beta portfolios

Another factor trading firms consider is their exposure to **beta** (denoted $\beta$), which is their relative risk explained by the broader market. However, no creative portfolio moves perfectly in line with the market, so we explain the rest of the risk with **idiosyncratic return** (denoted $\epsilon$) which are fluctuations in returns not explained by the broader market

If your portfolio has a positive beta greater than 1, it means when the market value has increased by 1%, your portfolio's value increases by more than 1%. Similarly, if the market value has gone down by 1%, the portfolio value has also gone down by more than 1%. If you portfolio has a positive beta less than 1, it means your market returns are less positive and less negative compared to the market's ups and downs. What happens if your beta is zero?

When the beta is zero, it means one of two things: your portfolio is not correlated with the market at all and/or your stocks have constant return. With /r/wallstreetbets making some crazy moves in the past year, the price of GME and AMC do not move according to the broader market. When the broader market value has increased by 1%, GME goes up by 69% and AMC decreases by 420% - it's hard to draw correlations between the performance between that of the market and the meme stocks. This means there is a small beta (we can consider it zero) but a large *idiosyncratic return* which explains its volatility. In the other case, we really indeed have zero volatility - you can think of this as putting your money in the bank to accrue interest at a rate locked in for the next few years. Your returns are fairly constant and it doesn't change depending on the stock market.

Our observations can be formalized in the following way:

**Returns for single stock:**

$$
r = \alpha + \beta m + \epsilon \quad \alpha,\beta,\epsilon \in \mathbb{R}\\
$$

**Returns for multiple stocks:**

$$
r = \sum_i^N \alpha_i + \beta_im + \epsilon_i
$$

where $r$ is the return of the portfolio and $m$ is the market return. Since we only have a single correlated factor to our portfolio in this equation (being $m$), we call this a (linear) **single-factor model**.

### Hedging

You've heard the word in "hedge funds", "hedge your bets", etc. Applied to the context of our portfolio, it means we want to remove our exposure to risk in some aspect. In the above model, we can hedge the market. In the case of a catastrophic correlated crash in the market, we want our trading firm to continue existing[^2]. To do this, we want our portfolio to have a beta of 0.

### Multi-factor model

We just considered the single factor model to describe the characteristics of return for our portfolio. We can do better by decomposing the market into a bunch of smaller factors. To be precise, we can model it as follows:

**Returns for single stock:**

$$
r = \alpha + \beta^Tf + \epsilon \quad\alpha,\epsilon \in \mathbb{R}, \beta, f\in \mathbb{R^m}
$$

**Returns for multiple stocks:**

$$
r = \sum_i^N \alpha_i + \beta_i^Tf_i + \epsilon_i
$$

where $b$ is the vector of **loadings** of a stock to a vector of factors $f$. Selecting the factors that best explain the returns of a portfolio is an art, and there are many ways to do it.

### Investing in high beta stocks

You would think that by investing in high beta stocks like TQQQ or some crazy growth stocks, you would have higher expected returns. This is actually shown historically to be incorrect! If we consider beta itself to be a factor in a multi-factor model, we will often times find the expected return of the beta factor to be negative! This means exposure to more volatile stocks has historically led to worse performance. Another reason to dump your money in S&P500 and forget about it, I guess.

## Evaluating Performance

There are many ways to think about the performance of a portfolio. One simple way is to just consider the expected returns of a portfolio, but that would fail to account for the potential volatility that may wipe out your firm. Most of the time, people use **sharpe ratios** to evaluate their performance, which is roughly:


$$
S = \frac{E(r) - R_b}{\sigma_r}
$$

where $R_b$ is some risk free return (i.e. putting it in a bank account to accrue interest) and $\sigma_r$ is the volatility of our portfolio. If we have high returns but we're living in [Zimbabwe](https://en.wikipedia.org/wiki/Hyperinflation_in_Zimbabwe) and using zimbabwe currency to measure our performance, maybe it's because of the hyperinflation, not because we're good portfolio managers. This corresponds to an extremely high risk free return, potentially making our sharpe ratio negative. If we're a US-based hedge fund that only invests in meme stocks, return may be very high but the volatility would be so high that no institutional investors would like to work with us. This corresponds to a very high volatility, making the sharpe ratio smaller.

One common way to size our positions for our portfolio after we've established a reasonable $\alpha$ (expected returns) and $\sigma^2$ (variance) is by **proportion of alpha**:

$$
\text{position} \propto \alpha
$$

which seems very simple. The more money we make or lose, the bigger the magnitude we go long or short scaled linearly. One may think this is myopic and should take into the volatility of the asset, so some have tried **mean variance**:

$$
\text{position} \propto \alpha/\sigma^2
$$

However, it has a couple of shortcomings and has worse simulated sharpe ratio compared to the dumb proportion strategy. Without getting into the details, the reason mean variance fails is because the estimation error of the expected returns can cause drastic changes to the positions of a portfolio, which is detrimental to performance.

## Attributing Performance

At the end of the day, we concretely see our PnL. How do we attribute our performance to individual factors?

$$
\text{PnL}(T) = \sum_{t=1}^T [\text{idioPnL}(t) + \sum_f \text{factorPnL}_f(t)]
$$

Usually, you'd like to plot the cumulative pnl for each factor and idiosyncratic returns to have a better idea which factors are performing well and which ones aren't. If some factors are doing badly, you would want to hedge more against the factors. If your idiosyncratic returns are doing badly, then that requires some more thought.

For idiosyncratic returns, you're basically betting on the stock market - you're guessing which stocks will outperform the markets it belongs to. Well, how do you quantify how well you're betting on the stock market? It roughly is a combination of **selection, sizing and timing**, and are intrinsically harder to quantify.

## Conclusion

When it comes to building a portfolio that makes you money, there's a whole field of mathematics powering it. What I've illustrated above only scratches the surface - in the subsequent posts we'll be looking more in depth into the modelling process, performance attribution, and other fun stuff!



[^1]: You would actually buy avocado *futures* - which allow cash settlement - so you wouldn't have to sell them to make your profit.
[^2]: But it's also important to note that if the market does really well, we don't reap any of the benefits if we hedge!

<script src="https://utteranc.es/client.js" repo="OneRaynyDay/oneraynyday.github.io" issue-term="pathname" theme="github-light" crossorigin="anonymous" async> </script>
