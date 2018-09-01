---
layout: post
title: "Going Beyond rand() for Monte Carlo Sampling"
description: "How the sampling method affects the speed of Monte Carlo integration"
date: 2018-06-24
tags: [monte, carlo, random, sampling, math, graphics, rendering]
comments: true
---

This articles assumes basic knowledge of calculus, random numbers, and a bit of
probability. It may be helpful to know the role of sampling in the context of
graphics and rendering.

_Note:_ if you spot any errata or think something can be explained more
clearly, please let me know in the comments section below. Fixes and
suggestions will be credited at the bottom.

## Introduction

You should be familiar with functions, integrals, and how you integrate a
function. Imagine a function with a complex integral. It may be difficult and
expensive to calculate, and due to these constraints, it may be more desirable
to instead calculate an approximation of the integral rather than analytically
compute the exact answer.

One such method that you can use for this is called Monte Carlo integration,
which uses sampling methods to estimate the value of an integral. Monte Carlo
integration has become particularly popular for rendering problems, as it
provides a cheaper way for computers to compute radiance values and simulate
light for the purposes of ray-tracing and path-tracing. This technique is
useful for approximating any complex integral, and it is not limited to
applications in rendering.

### Monte Carlo Integration

Monte Carlo integration computes the value of an integral by randomly sampling
points of some function (the integrand) $$ f(x) \$$. In other words, we will
generate random numbers, use those as the input for $$ f(x) \$$, and use the
resulting values to estimate the value of the integral.

You are probably wondering how exactly we can go from random values of $$ f(x)
\$$ to estimating $$ \int\_{a}^{b} f(x) \$$. Before I explain that, let's add
some intuition to the idea with a classic Monte Carlo example.

#### Calculate $$ \pi \$$ with Monte Carlo

Imagine a dartboard that's 2 meters by 2 meters. In the dart board, you have a
perfect circle with four points tangent to the edges of the square. Thus, the
circle has a diameter $$ d = 2 \$$, and a radius $$ r = \frac{1}{2}d = 1 \$$.
We know analytically that the area of this circle is $$ \pi \$$ because the
area of a circle $$ A = \pi r^2 \$$. What if we didn't know that? What if we
don't know the value of $$ \pi /$$?

We can randomly throw darts at the dart board, and record the number of darts
that fall within the circle, and the number of darts that are in the dartboard
but fall outside of the circle. Let's call this ratio of the number of darts
that fall inside the circle to the total number of darts thrown $$ i \$$. We
can throw any number of darts.

Now what? This ratio we have gives us the fraction of the area inside the
dartboard that comprises the circle's area. We know the total area, because we
know the size of the dartboard. We can get the area of the circle by
multiplying the ratio $$ i \$$ with the area of the dartboard, giving us the
area of the circle, which is $$ \pi \$$. As we increase the number of darts
thrown, this ratio gets more an more precise, until eventually, an infinite
number of darts cover the dartboard, giving us the exact ratio between the
amount of space the circle occupies and the total area of the dartboard.

So we can figure out the area of a circle (or really any arbitrary shape)
within some well-defined area that, and we also know that taking more samples
(analogous to throwing a dart) leads to a more accurate answer.

#### Rejection Sampling

Let's bring the idea back to integration. Imagine some finite domain and some
function $$ f(x) \$$. Within this finite domain, we are going to select random
values, and see whether the points selected fall within a function or outside
of the function. Given the size of the finite domain, we can use the ratio of
points that fall within the function to the size of the containing area to
estimate the area that falls under the curve of the function. An integral is
the area under a curve for a two dimensional function, so through this method,
we can estimate the integral of some complex function. This method is called
rejection sampling. You can imagine taking a sample, and checking whether it
falls within the founds of the function or is "rejected" by it.

#### Monte Carlo Integration

We'll consider "standard" Monte Carlo integration to be Monte Carlo integration
with uniform random sampling. This simply means that when we are choosing random
numbers, we are using some random number generator in which the probability of
a number being chosen in the domain is equal for every number in the domain (hence
the _uniform_ part).

Let's define some integral

$$ I = \int f(x) dx $$

This integral can be approximated by the following summation:

$$ I = \frac{1}{N} \sum\_{i = 1}^{N} f(x) $$

That's it! You calculate the values for a function at a bunch of different points
within the domain, and then you average out the values. This seems simple, but it
has powerful implications. This provides an easy way to create computational programs
that can evaluate integrals through raw power, rather than being forced to design a
system that can evaluate complex integrals. This method also generalizes to any number
of dimensions, and increasing the number of dimensions does _not_ slow down the method.
In fact, the rate of convergence for Monte Carlo integration with uniform sampling is
always $$ \frac{1}{\sqrt{N}} \$$ regardless of the dimensionality of the integral.

Another nice thing about Monte Carlo is that it's progressive. You can think of $$ N \$$,
or the number of sampled points as a quality control knob. The higher it is, the more
accurate your answer, but the longer it will take to compute. A lower value will lead
to a faster answer, but an answer that's a little less accurate.

Finally, Monte Carlo integration has once issue: it's non-deterministic. This means that
if you calculate the same integral with the same number of samples multiple times,
you're not guaranteed to get the same answer, even though it will always converge
_towards_ the same answer (which is the true value of the integral).

## Sampling Methods

Perhaps you are wondering why we choose random numbers for Monte Carlo integration.
It has some downsides: non-determinism, and a convergence rate that leaves something
to be desired. It turns out that using random uniform sampling does not yield the
fastest convergence for Monte Carlo integration. There other multiple sampling methods
that can lead to better performance and faster estimation. Uniform random sampling is
easy to implement, however, and has been proven to converge towards the right answer, so it's a
popular choice.

In order to evaluate sampling methods, you need to understand what makes a
good sampler, and what traits we want to see in our samplers that will likely lead to
fast and accurate estimates of integrals.

### Jittered/Stratified Sampling

### N-Rooks/Latin Hypercube Sampling

### Poisson Disk Sampling
