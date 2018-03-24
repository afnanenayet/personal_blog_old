---
layout: post
title: "Stumbling Upon Pi"
description: "Calculating the value of pi via monte carlo simulation"
date: 2018-03-23
tags: [monte, carlo, probability, random, google, interview, question, pi]
comments: true
---

There are a number of methods that you can use to calculate the value of pi.
One such way is through something called a monte carlo simulation.

## What is the Monte Carlo method?

First, you have to understand what the Monte Carlo method actually is. It
sounds fancy but in reality it's really quite simple. The Monte Carlo method
allows us to solve problems that involve random variables by running
simulations where those random variables are replaced by actual randomly
generated numbers. You want some sort of random number generation that actually
gives good results, and you run these models with lots of these randomly
generated numbers to get a very good approximate answer. This technique is used
for weather forecasting and market analysis with great success. Anywhere you
have a model that has some sort of dependence on random variables, you can try
to approximate the model by trying it out with a lot of actual random numbers.

## Where does pi come in?

Let's suppose we're trying to calculate the area of a circle. There is a pretty
simple equation to do that, provided we know the radius, $$ r $$, of the circle.

$$
\begin{equation}
    A = \pi r^2
\end{equation}
$$

If $$ r = 1 $$, then $$ A = \pi $$. So it seems that if we could calculate the
area of a circle with a radius of 1, we get the value of $$ \pi $$. What a
great start. Of course, if we don't know the value of $$ \pi $$, how do we
calculate the area of the circle?

### Approximating the area of a circle

This is where the Monte Carlo method comes into play. We need some way to
approximate the value of a circle without actually knowing the value of
$$ \pi $$.
