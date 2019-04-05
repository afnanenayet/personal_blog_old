---
layout: post
title: "A Memory and Space Constant Shuffling Algorithm"
description: "Andrew Kensler's hashed permutation technique"
date: 2019-04-05
tags: []
comments: true
---

Andrew Kensler, from Pixar, introduced an interesting technique for generating
the permutation of an array in his 2013 paper,
[Correlated Multi-Jittered
Sampling](https://graphics.pixar.com/library/MultiJitteredSampling/paper.pdf).

Firstly, let's look at the naive way of generating a permutation. You construct
an array of elements from $0 \cdots n$, and then you randomly shuffle them.
Then, your resulting array (let's call it `A`), will have the permuted value
for `i` at `A[i]`.

```python
from random import shuffle
n = 10

permutation = list(range(n))
permutation = shuffle(permutation)
```

The bright side of this is that it's really easy to implement and fairly easy
to access. You simply subscript the array at whichever number you want to
permute and you get the permutation for that number. The downside is that it's
`O(n)` for space complexity and `O(n)` for time complexity. At the very
least, you need to create an array of `n` elements, yielding an `O(n)` space
complexity, and then generate the array, which is `O(n)` time, and then shuffle
the array, which is also `O(n)` with the Fisher-Yates algorithm, which the
`shuffle` method from Python uses.

Kensler's paper introduces a way to turn this into `O(1)` for both space and
time through some clever hashing techniques. He mentions some prior work in AES
which uses hashed permutations, and notes that any hash function that is
reversible must be a permutation so long as it remains within a defined domain.
Why? Consider that a hash is (ideally) a 1-1 mapping of elements. If we are
constrained to a domain, then that means that every element in a domain maps to
another element within the domain, and thus we end up with a unique mapping of
elements to other elements. Because the domain of the input set and output set
are the same, we have a random shuffling of the input set.

In Kensler's paper, he defines a set of operations that are reversible in any
domain that is a power of 2. Most of these are trivially reversible, and don't
forget that shifting a number to the left by one bit simply amounts to squaring
it, and shifting to the right by one bit is the same as taking the truncated
square root.

```c
hash ^= constant;
hash *= constant; // if the constant is odd
hash += constant;
hash -= constant;
hash ^= hash >> constant;
hash ^= hash << constant;
hash += hash << constant;
hash -= hash << constant;
hash = (hash << constant) | (hash >> wordsize_constant);
```

Of course, you may be a little put off by the constraint that the domain must
be a power of two, but Kensler shows how we can mitigate that to operate
within arbitrary domains using cycle walking. Cycle walking is a technique in
cryptography where you basically just repeatedly encrypt information until it
falls within an acceptable range. In this case, we can continue applying
permutations to a number until it falls within the proper range.

Let's take a look at the actual permutation function.

```c
/*!
 * \brief Permute a number
 *
 * \param i The number to permute/the index of the permutation vector
 * \param l The desired size of the permutation vector
 * \param p The seed of the shuffle
 */
unsigned permute(unsigned i, unsigned l, unsigned p) {
  unsigned w = l - 1;
  w |= w >> 1;
  w |= w >> 2;
  w |= w >> 4;
  w |= w >> 8;
  w |= w >> 16;

  do {
    i ^= p;
    i *= 0xe170893d;
    i ^= p >> 16;
    i ^= (i & w) >> 4;
    i ^= p >> 8;
    i *= 0x0929eb3f;
    i ^= p >> 23;
    i ^= (i & w) >> 1;
    i *= 1 | p >> 27;
    i *= 0x6935fa69;
    i ^= (i & w) >> 11;
    i *= 0x74dcb303;
    i ^= (i & w) >> 2;
    i *= 0x9e501cc3;
    i ^= (i & w) >> 2;
    i *= 0xc860a3df;
    i &= w;
    i ^= i >> 5;
  } while (i >= l);
  return (i + p) % l;
}
```

_This was taken from Kensler's paper, the comments added to explain the function
signature are mine._

This looks rather daunting, with a lot of bitwise operations, but remember that
the entire thing is comprised of operations we just discussed are being
reversible. This means that this is a hash operation that is a permutation, so
each number will unique map to another number within the smallest power of two
domain that is larger than `l`.

The body of the function inside of the do-while loop is basically just applying
a permuted hash repeatedly until our number is less within the domain, which we
define with `l`. `p` can be more or less treated as a random seed, it lets us
apply some arbitrary offset which retaining the uniqueness property of a
permutation. It's fairly self explanatory: apply some offset and modulo it
within the domain of $0 \cdots l$, and because the input domain is the same as
the output domain, we retain the 1-1 mapping and always get a proper permutation.

Using this function gives you a very low overhead way to generate particular
permutations or shuffles on the fly with `O(1)` space and time complexity.
