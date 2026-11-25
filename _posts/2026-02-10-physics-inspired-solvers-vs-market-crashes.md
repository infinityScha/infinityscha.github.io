---
layout: post
title: "Physics-Inspired Solvers vs. Market Crashes"
thumb: /images/mvar/thumb.svg
summary: "I ported an optimizer from biophysics to portfolio risk. The textbook solver won, and a crash is why."
---

In my PhD I minimize membrane energies with FIRE2, an inertial optimizer from molecular dynamics. It gives the solver a mass and a velocity, so it can fly over small ripples instead of getting trapped in them. I wanted to see if it works for portfolio optimization too.

The target was Modified Value-at-Risk. It corrects the Gaussian quantile with skewness and kurtosis, which are cubic and quartic in the weights, so I expected a rugged, non-convex surface.

### The build

FIRE2 inside an Augmented Lagrangian loop, and Numba-compiled gradients for the O(N<sup>4</sup>) kurtosis tensor.

### The result

I used 45 large S&P 500 stocks from 2007 to 2009, at 99% confidence. FIRE2 reached the same minimum from all 5 random starts. SLSQP did in 3 of 5 and stopped about 0.4% higher in the other two. But SLSQP took about 0.4 s per run and FIRE2 about 15 s.

The reason is that in a crash everything falls together:

{% include mvar-figure.html %}

The first principal component carries 54% of the variance in the crisis, against 41% in 2017–2019. The effective number of independent factors drops from 5.6 to 3.4. At the 1% tail the problem is close to convex, and a quasi-Newton method does not need inertia.

So the physics approach is overkill for systemic events. The gradient engine is the part worth keeping, and any gradient-based solver can use it. This is a small test: one set of stocks, one crisis, in-sample.

Code: [github.com/infinityScha/inertial-risk-optimizer](https://github.com/infinityScha/inertial-risk-optimizer)
