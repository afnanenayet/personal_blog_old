---
layout: post
title: "Explaining Gradient Descent"
description: "A cursory explanation of gradient descent and stochastic gradient descent"
date: 2017-09-28
tags: [machine, learning, gradient, descent, loss, function, math]
comments: true
---

## A classic statistics problem

In machine learning, problems are often framed as optimization problems. For example,
let us take one of the simplest applications of supervised learning: the linear
regression. It's conceptually an easy problem - given a set of data points,
create a function $$ f = ax + b $$ that best approximates these points. If you
ever took statistics or have used excel, this is the line of best fit.
Conceptually, this operation doesn't seem too difficult. Note that this is a
supervised learning problem - we need to have a set of observed data to "train"
from. We use this data to create the line of best fit, which we can then subsequently
use to make predictions given new parameters.

## Framing it as a machine learning problem

### Defining a linear regression

Let's rewrite the equation $$ f(x) = ax + b $$ as $$ f(x) = a_1 x + a_0 $$. We have
two weights that we need to define, $$ a_0 $$ and $$ a_1 $$, that correspond
to the features $$ x_0 $$ and $$ x_1 $$. $$ x_0 = 1 $$ and is kind of a
"dummy variable" because we need to fit a linear regression which has the
offset $$ a_0 $$, there isn't any actual observation associated with $$ x_0 $$
and it has no measured effect on the final line of best fit.

Our data is defined as a matrix $$ X $$ where each row of the matrix represents
the readings for the $$ i^{th} $$ observation. Each row, $$ x^{(i)T} $$, is the
set of observed variables for a particular observation, which means it must have
our feature and the observed output. Note that it is transposed because $$ x^{(i)} $$
is a vector with the recorded features for that observation. We have another vector
$$ Y $$ that holds the actual observations for $$ X $$, so the $$ i^{th} $$ element
in $$ Y $$ corresponds to the $$ i^{th} $$ row of $$ X $$.

Our objective is to create some function $$ f $$ in the form $$ f(x) = a_0 + a_1 x $$
that best approximates the data in $$ X $$ and $$ Y $$. This means that we need to
pick $$ a_0, a_1 $$ such that $$ f $$ best approximates _all_ of the data in $$ X $$.
So how do we pick the best $$ a_0, a_1 $$?

### The loss function

Suppose we randomly guess $$ a_0, a_1 $$. We want to know how far off from the actual
data we are, we need a quantitative way to determine how wrong we are. We can measure
the "wrongness" of our function with something called a loss function. A loss function
is a function that provides a measure of your wrongness, and our objective is to
minimize it as best as possible. There are several loss functions you can use, a very
popular loss function is called mean-squared error (MSE).

We define $$ MSE $$ as follows:

$$
\begin{equation}
MSE = \frac{1}{n} \sum_{i = 1}^n (y^{(i)} - f(x^{(i)}))^2
\end{equation}
$$

In essence, we are taking every row in $$ X $$, making a prediciton with our function
$$ f = a_0 + a_1 x $$, taking the difference between the actual $$ y $$ value and
our prediction $$ f(x^{(i)}) $$, then squaring the difference. We do this for every
row, then we divide by $$ n $$ (the number of data points we have). This gives us
the average squared error in the data set, and lets us quantify how good our
line of best fit is. Of course, we want to minimize our loss function.

We can find the minimum of a quadratic function by finding the root of the
derivation. Note that it has to be a concave down function, otherwise the
root will give us the maximum.

Let us define our loss function $$ L(a) $$ as $$ MSE $$ and find the minimum.

$$
\begin{align}
L(a) &= \frac{1}{n} \sum_{i = 1}^n (y^{(i)} - f(x^{(i)}))^2 \\
L(a) &= \frac{1}{n} \sum_{i = 1}^n (y^{(i)} - (a^T x^{(i)})^2 \\
L'(a) &= \frac{1}{n} * 2 (y^{(i)} - a^T x^{(i)}) * -x^{(i)} \\
L'(a) &= -\frac{2}{n} (y^{(i)} - a^T x^{(i)}) x^{(i)} \\
0 &= -\frac{2}{n} (y^{(i)} - a^T x^{(i)}) x^{(i)}
\end{align}
$$

We have our derived function, solving for $$ a $$ will be left as an
exercise to the reader.

## Gradient Descent

With our defined loss function, how do we figure out how to guess $$ a $$ in
an iterative manner? We could start with some random vector for $$ a $$ and
then increment the variables a little, trying to lower $$ L(a) $$ with every
step until we converge on the minimum (so $$ L(a) $$ cannot be minimized
any further). This is the intuition behind gradient descent, except
gradient descent has a clever way of incrementing the new $$ a $$ values so
that every time gradient descent iterates, $$ L(a) $$ is guaranteed to
decrease.

In gradient descent, take our vector $$ a_t $$ where $$ t $$ is our current
iteration. With each iteration of gradient descent, we set $$ a_{t+1} $$
equal to some new value as defined below:

$$
\begin{align}
a_t &= a_{t-1} - \alpha f(a_{t-1})
\end{align}
$$

Gradient descent allows us to iteratively minimize the loss function - in other
words, this is how we find the value with the least amount of error. Intuitively,
imagine a ball rolling down a hill. The hill is drawn by the loss function, and
we want to reach the bottom. Because we are subtracting by a ratio defined by
the slope of the hill, with a convex function, we should be able to optimize
and find the minimum. If we don't have a strictly concave down function,
there is a chance we will not be able to find the minimum. Additionally,
note the $$ \alpha $$ term. This is the step size, this is a constant that
we define which will affect how fast our gradient descent mechanism rolls
the ball down the hill. If we keep the step size small, it could take a while
until we reach the bottom of the hill, it might also get stuck in a local minima.
If the step size is too large, it may oscillate around the minimum but never
quite converge.

### Optimize for big data (stochastic gradient descent)

Gradient descent can be an expensive process - imagine our $$ n $$ is extremely
large (this is the definition of big data). If $$ n $$ is really big, then
every iteration of gradient descent will be extremely expensive. In practice,
we must balance the accuracy of our models with computation time. A lot of
models that use massive amounts of data mitigate this with a loss function
called stochastic gradient descent.

The main intuition behind stochastic gradient descent is that the model is
trained and tested on a small subset of the data at a time. This way, you
get a rougher approximation of the optimal value(s), but it is much faster.
Because gradient descent is an approximation itself, the penalty in accuracy
brought on by stochastic gradient descent isn't big enough to outweigh the
performance gains that are brought on by operating on only a batch of the
data at a time.

If you're using stochastic gradient descent, you'll want to randomly shuffle
the data, and perform the operation on that randomly shuffled mini-batch.

