---
layout: post
title: "Stumbling Upon Pi"
description: "Calculating the value of pi via monte carlo simulation"
date: 2018-03-23
tags: [monte, carlo, probability, random, google, interview, question, pi]
comments: true
---

There are a number of methods that you can use to calculate the value of pi.
We are going to explore an option

## Understanding the Monte Carlo method

First, you have to understand what the Monte Carlo method actually is. It
sounds fancy but it's really quite simple. The Monte Carlo method
allows us to solve problems that involve random variables by running
simulations where those random variables are replaced by actual randomly
generated numbers. Running these models with lots of randomly
generated numbers tends to get a very good approximate answer. This technique is used
in weather forecasting and market analysis with great success. Anywhere you
have a model that has some sort of dependence on random variables, you can try
to approximate the model by trying it out with a lot of actual random numbers.

## Finding pi

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

Imagine you have a dartboard. This dartboard is a `2x2` dartboard with a
circle with radius $$ r = 1 $$ right in the middle. This means that the
edges of the square touch the circle.

We know the total area of the square. Each side is $$ 2 $$, $$ 2^2 = 4 $$.
If we can figure out the ratio of the area of the circle to the area of the
square, we can multiply the ratio with the total area of the square to
get the area of the circle. We established earlier that the area of the circle
is equal to $$ \pi $$.

In order to actually get this ratio, randomly throw a lot of darts at this
imaginary dartboard. In this case, we will define a "random throw" as a throw
that has an equal chance of landing anywhere on the board. Suppose the number
of total throws is $$ N $$, and the number of darts that fall within the
circle, or the number of hits, is $$ h $$. We'll call the total area of the
square $$ A $$. We can approximate the ratio of the area of the circle to
the area of the square with $$ \frac{h}{N} $$. The area of our circle is
$$ \frac{h}{N} A $$. Thus, $$ \pi = \frac{h}{N} $$.

### Computing the approximation

It takes a lot of time to throw a lot of darts randomly at a dart board,
especially if you're looking at values of $$ N $$ that are greater than
1000, or maybe even a million.

Luckily, you can offload this to a computer, to even a simple Python program,
and get your very own approximation of pi.

First, we need to define a dart throw. If we were to define our dart board as
a plane, our dart would have an $$ x $$ and a $$ y $$ coordinate that falls
within the bounds of our dart board. Let's say the four corners of our dart
board are $$ (-1, -1), (-1, 1), (1, 1), (1, -1) $$. Then our $$ (x, y) $$
must fall within these bounds. We are gratuitously assuming that in this
scenario, the dart thrower has the ability to randomly distribute throws, but
only within the bounds of the board.

In python, generating $$ (x, y) $$ looks something like this:

```python
import random

# constants
UPPER = 1
LOWER = -1

x = random.uniform(LOWER, UPPER)
y = random.uniform(LOWER, UPPER)
```

Next, we need to determine whether the dart fell in the circle or not. We have
a very useful equation that can help us figure this out given a coordinate set.
The equation to draw a circle is:

$$
\begin{equation}
    \sqrt{x^2 + y^2} = r
\end{equation}
$$

We can see if the dart fell within the circle if

$$
\begin{equation}
    \sqrt{x^2 + y^2} < r
\end{equation}
$$

is true.

In practice, we would run a loop, looping $$ N $$ times, and generate a
random set of coordinates each time, checking and recording whether those
coordinates fall within the bounds of the circle. We then return the ratio of
the area of the circle to the area of the square multiplied by the total area
of the square to give us the area of the circle, which is equal to pi. This
gives us an approximation of pi.

Here's the code in Python 3:

```python
import random
from math import sqrt

# constants
LOWER = -1
UPPER = 1

def monte_carlo_pi(N: int) -> float:
    """ uses the monte carlo method to approximate the value of pi using the
    techniques discussed in an awesome blog post
    N: the number of iterations to use to approximate pi
    returns: the approximate value of pi
    """
    hits = 0

    for _ in range(N):
        x = random.uniform(LOWER, UPPER)
        y = random.uniform(LOWER, UPPER)

        if sqrt((x ** 2) + (y ** 2)) < 1:
            hits += 1
    # (hits / N) is the ratio of the area of a circle to the area of the square
    # that holds it.
    # UPPER - LOWER gets the length of a side of the square, and squaring it
    # yields the area of the square.
    # multiplying these two numbers together yields the area of the circle,
    # which is equal to pi
    return (hits / N) * ((UPPER - LOWER) ** 2)
```