---
layout: post
title: "All possible encodings from a numerical mapping"
description: "Explaining a Facebook interview that utilizes dynamic programming"
date: 2018-03-12
tags: [facebook, interview, question, prep, dynamic programming, subproblems, recursion
]
comments: true
---

I haven't done interview prep in a while, and I decided to get back into it
after I saw a practice problem that caught my eye. I got it from
[this](dailycodingproblem.com) mailing list.

## The problem

Suppose we have a mapping of letters to numbers, or an encoding, such that
$$ a = 1, b = 2, \cdots z = 26 $$.

Given a string of digits, find the number of possible ways that this message
could be decoded.

For example, `111` has three different mappings:

1.

1 | 1 | 1
--- | --- | ---
a | a | a


2.

11 | 1
--- | ---
k | a

3.

1 | 11
--- | ---
a | k

## The approach

In an interview, you should never begin by coding. Think through the problem
first, and identify a strategy. You want to have a high level overview of what
you're going to do before you do it, otherwise you run the risk of going down
a tangent, and then realizing you got too focused on little details and
ended up with a suboptimal, or even totally incorrect approach.

You should also ask your interviewer questions. For example, for the sake of
our problem, are we guaranteed valid strings? Should we worry about the total
length of strings? What do we do when passed an empty string? What do we do
when we receive a string without numbers or that is a valid string but isn't
valid for our test cases? Can there be zeros before or after the "main"
sequence?

For the purposes of this exercise, let's assume we're getting valid strings,
and that all we have to worry about is the argument, not parsing input, or
worrying about getting the result back to the user. Don't worry about the empty
string. Usually your interviewers
aren't out to trip you up on minutiae, and all of my interviews have generally
been more focused on the actual algorithm and how well I think out my answer
rather than the smaller details.

One of the first things that came to mind was that this problem can be broken
down into subproblems, and that each letter is restricted to two digits at
maximum. That is, because `z = 26`, and all letters are between `[a, z]`
(inclusive). This means, that every single digit must correspond to a letter,
and that every pair of digits _can_ correspond to a letter.

### The recursive approach

The first thing that came to mind for me is that this problem can be broken down
into recursive subproblems.

Suppose $$ s $$ is a string, and $$ s_i $$ represents the string from elements
0 to the $$ i^{th} $$ element.

The number of ways $$ s_i $$ can be interpreted is equal to the number of ways
$$ s_{i - 1} $$ can be interpreted, unless the last two characters form a letter
(that is, $$ 0 < s_{i - 1} * 10 + s_{i - 2} \leq 26 $$). Then we have the number
of ways that $$ s_{i - 1} $$ can be interpreted and all the ways $$ s_{i - 2} $$
can be interpreted.

In recursion, it's really important to sort of just trust the recursive property.
It can be hard to wrap your head around, and sometimes you might even overthink
it and convince yourself you're wrong, but as long as you have your subproblems
defined correctly and good base cases, you will be fine.

Our base cases are $$ s_0 $$ and $$ s_1 $$. They can both be interpreted in only
one way.

```python
def recursive_count(s: str) -> int:
    """
    Determines the number of ways a string or array of digits can be decoded
    given the decoding rules provided by the problem.
    s: an array of single digits integers (every digit is between 1 and 26,
    inclusive)
    :return: the number of ways `s` can be decoded
    """
    if not s or len(s) == 1:
        return 1

    count = 0

    if int(s[-1]) > 0:
        count = recursive_count(s[:-1])

    prev_two = (int(s[-2]) * 10) + int(s[-1])

    if 0 < prev_two < 27:
        count += recursive_count(s[:-2])
    return count
```

This is not the most efficient way to go about this, however. We can optimize
this further by memoizing. You may notice that this problem has a very
familiar structure. We can break it down as such:

Suppose $$ f(x) $$ is the function that returns the number of ways some string
$$ x $$ can be decoded. Then $$ f(x) = f(x_{n - 1}) + f(x_{n - 2}) $$. Of course,
the second part of that statement is conditional, we have to check whether two
letters can be decoded to a letter (e.g. are they less than or equal to 26),
but this is very close to the recursive equation for the Fibonacci sequence.

### Memoizing

We can improve the efficiency of this solution by caching the results of each
subproblem in an area. This is called
[memoizing](https://en.wikipedia.org/wiki/Memoization) (_not memorizing!_).

In this situation, suppose we have some array `A`. We will enforce that `A[i]`
returns the number of ways that the subset of the string from the first
character up to the $$ i - 1^{th} $$ character can be decoded. If we see that
two characters can be decoded as a character, we will add the number of ways
that the substring from 0 to $$ i - 2 $$ can be interpreted (that's just
the string if we remove the pair of characters we just looked at).

In a way, we're almost looking at paths. How many ways can we traverse the
string when we can either traverse one letter at a time, or under certain
conditions possibly skip a letter (since when you interpet a pair of letters,
the next letter you decode is the one after that pair).

```python
def dp_count(s :str) -> int:
    """
    Determines the number of ways a string or array of digits can be decoded
    given the decoding rules provided by the problem.
    s: an array of single digits integers (every digit is between 1 and 26,
    inclusive)
    :return: the number of ways `s` can be decoded
    """
    # add an extra element because cache[len(s)] should return our answer
    cache = [0] * (len(s) + 1)
    cache[0], cache[1] = 1, 1

    for i in range(2, len(s) + 1):
        if int(s[i-1]) > 0:
            cache[i] = cache[i-1]

        prev_two = (int(s[i-2]) * 10) + int(s[i-1])

        if 0 < prev_two < 27:
            cache[i] += cache[i-2]
    # this simply returns the last element in the array
    return cache[-1]
```

Now we have a solution with $$ O(n) $$ space and time complexity, which is as
efficient as you can get.

A nice little trick you can use to determine what the best possible efficiency
for a particular problem is to think about how you would go about solving the
problem. To me, it makes sense that this is an $$ O(n) $$ problem because we
have to look at all of the elements in the string/byte array at least once if
we want to find out how many ways there are to decode it. We also know that
we only have to look at up to two characters at any point because the upper
bound for a decoded letter is two digits, but it has to be at least one digit,
which is why we essentially have a sliding window of two characters.