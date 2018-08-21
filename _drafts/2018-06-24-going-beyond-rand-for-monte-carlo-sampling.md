---
layout: post
title: "Going Beyond rand() for Monte Carlo Sampling"
description: "Why we can do better than just using canonical random numbers for convergence"
date: 2018-06-24
tags: [monte, carlo, random, sampling, math, graphics, rendering]
comments: true
---

## Introduction

Monte Carlo simulations involve generating many random numbers to estimate
some value. The way these random numbers are generated can have a big effect
on how quickly a Monte Carlo integration converges to the actual solution.
In this article, we will explore and describe a few methods that can be
used to reduce variance and make convergence faster when trying to estimate
the value of an integral with the Monte Carlo method. Many Monte Carlo methods
only use the normal random sampler, that is to say, every sample that's
generated is a pseudo-random number or some independent linear transformation
of one.

### Monte Carlo Integration

To understand the effect of sampling strategies on the convergence of a Monte
Carlo estimator, you first need to understand how you can estimate the value
of an integral with the Monte Carlo method. I'm not going to get too much
into the intuition behind how Monte Carlo works, because there are a lot of
articles out there that do an excellent job of explaining this.

Suppose we have some equation $$ f(x) \$$, and the integral of $$f(x) \$$ is
$$ F(x) \$$.
To estimate $$ F(x) \$$ over some defined, finite interval, we can use the Monte
Carlo estimate, which is defined as

$$ \sum\_{i = 1}^N \frac{f(x)}{pdf(x)} $$

This converges to the correct answer at a rate of $$ \frac{1}{\sqrt N} \$$.
This means that to reduce the error by a factor of two, we need to use 4
times as many samples. This may not seem optimal, but Monte Carlo estimation
has the distinct advantage of being independent of dimensionality. Regardless
of how many dimensions you're integrating over, the convergence rate will
remain the same.

### Other Useful Terms

(TODO)

- bias
- variance
