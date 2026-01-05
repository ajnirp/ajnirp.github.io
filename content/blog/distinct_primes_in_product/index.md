+++
title = "Leetcode #2521: Distinct Prime Factors of Product of Array"
date = "2026-01-04"
+++

A fun Leetcode problem I solved recently was [#2521](https://leetcode.com/problems/distinct-prime-factors-of-product-of-array/). In this blog post I'll discuss a couple of solutions to it.

## Problem statement

The problem statement simply asks us to find the number of **distinct prime factors** in the product of the elements of an array of positive integers. So for example the product of the elements of the array `[1, 2, 6, 7]` is `84`, whose prime factors are `[2, 2, 3, 7]`, which has `3` _distinct_ elements, so our answer should be `3`.

The array can have at most `10,000` elements and each element lies between `2` and `1000`.

## A first attempt

A useful observation here is that we don't even need to multiply together all the array elements. We're only trying to count the **distinct prime factors** of the product, so if we build up a set of prime factors for each element and union all of them together and find its size, we'd be done. This way we sidestep issues with loss of precision or numeric type overflows.

We could of course get around the latter restriction by using a language like Python or a language library that supports arbitrary-precision integers, but that seems like overkill when we already have the simpler idea discussed above.

How do we find the prime factors of each element of the array? My first thought was to build up a list of primes from `2` to `1000`, since that's the range the array elements lie in. Then we could check each number against the list of primes, like so:

```python,linenos
class Solution:
    def distinctPrimeFactors(self, nums: List[int]) -> int:
        def is_prime(n):
            return not any(n % i == 0 for i in range(2, n))
        PRIMES = [k for k in range(2, 1001) if is_prime(k)]
        seen = set()
        for num in nums:
            for prime in PRIMES:
                if prime > num:
                    break
                if prime not in seen:
                    if num % prime == 0:
                        seen.add(prime)
        return len(seen)
```

In this code:

* To construct the list of primes we use a naive algorithm that simply marks a number `k` as prime if it has no integer factors in the range `[2, k-1]`. There are of course many better, more sophisticated algorithms to check primality. 

* The `if prime > num: break` clause on lines 11 and 12 lets us skip pointless checks for `num % prime == 0`, which can never be true if `prime` is greater than `num`.

This solution passes all testcases, but the runtime looks to be suboptimal.

<figure>
<img src="leetcode-2521-attempt1.png" alt="A first attempt at solving Leetcode #2521. The runtime only beats 21% of submissions.">
<figcaption>A first attempt at solving Leetcode #2521. The runtime only beats 21% of submissions.</figcaption>
</figure>

## Doing better

### The sieve of Eratosthenes

There isn't much to refine in the approach we used for our first attempt. We'll need to try a different approach.

When dealing with prime numbers it's natural to think of the [Sieve of Eratosthenes](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes#Overview). This is an algorithm to generate all prime numbers up to a given number, in this case `1001`. The simplest implementation of this algorithm looks like [this in pseudocode](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes#Pseudocode).

It turns out we can extend the sieve in a clever way — we can store the smallest prime factor for each number instead of a simple boolean value. I read about this approach in [this GeeksForGeeks post](https://www.geeksforgeeks.org/dsa/prime-factor/).

```python,linenos
N = 1000  # Upper bound on primes to generate, as per the problem statement
smallestPrimeFactor = [i for i in range(N+1)]
for i in range(2, N):
    if smallestPrimeFactor[i] == i:  # prime!
        for j in range(2*i, N+1, i):
            smallestPrimeFactor[j] = i
```

Why store the smallest prime factor for each number? Because now we have a quick way to factorize any number lying between {{ katex(body="1" )}} and {{ katex(body="N" )}}:

```python,linenos
# Returns the prime factorization of `num` in sorted ascending order.
def factorize(num: int, smallestPrimeFactor: list[int]) -> list[int]:
    prime_factors = []
    while num > 1:
        prime = smallestPrimeFactor[num]
        prime_factors.append(prime)
        num //= prime
    return prime_factors
```

It's straightforward to extend this function to instead dump all prime factors for a given number into a set external to the function. When we're done calling this function for every number in the array, we will have a set consisting of all of the distinct prime factors of the product of the entire array!

```python,linenos
distinctPrimes = set()
def factorizeAndUpdateDistinctSet(num: int, smallestPrimeFactor: list[int], distinctPrimes: set[int]):
    while num > 1:
        prime = smallestPrimeFactor[num]
        distinctPrimes.add(prime)
        num //= prime
```

And putting it all together, we have a working solution:

```python,linenos
N = 1000
smallestPrimeFactor = [i for i in range(N+1)]
for i in range(2, N+1):
    if smallestPrimeFactor[i] == i:  # prime!
        for j in range(2*i, N+1, i):
            smallestPrimeFactor[j] = i
distinctPrimes = set()
def factorizeAndUpdateDistinctSet(num, smallestPrimeFactor, distinctPrimes):
    while num > 1:
        prime = smallestPrimeFactor[num]
        distinctPrimes.add(prime)
        num //= prime
for num in nums:
    factorizeAndUpdateDistinctSet(num, smallestPrimeFactor, distinctPrimes)
return len(distinctPrimes)
```

...and it's much faster than before!

<figure>
<img src="leetcode-2521-attempt2.png" alt="Our runtime now beats 92% of all submissions.">
<figcaption>Our runtime now beats 92% of all submissions.</figcaption>
</figure>

### Optimization: skipping a few iterations

Every time we encounter a prime number, we "cross out" (set the corresponding index in `smallestPrimeFactor`) all of its multiples. To do so we'd move the index {{ katex(body="j" )}} from {{ katex(body="2i" )}} all the way up to {{ katex(body="N+1" )}} (upper bound exclusive) by steps of {{ katex(body="i" )}}, where {{ katex(body="N" )}} is the upper bound on primes that we want to generate.

But there's a nice optimization available to us. Consider a primes {{ katex(body="i" )}}. By the time our iteration variable {{ katex(body="j" )}} reached {{ katex(body="i" )}}, we had already processed _every_ multiple of _every_ prime that was smaller than {{ katex(body="i" )}}. For example:

* we had already processed every multiple of {{ katex(body="2" )}}, including {{ katex(body="2i" )}}
* we had already processed every multiple of {{ katex(body="3" )}}, including {{ katex(body="3i" )}}
* ...and so on!

To be precise, by the time {{ katex(body="j" )}} reaches {{ katex(body="i" )}}, we have already processed every multiple of {{ katex(body="i" )}} smaller than {{ katex(body="i \times i" )}}. We know for a fact that we haven't already processed {{ katex(body="i \times i" )}} itself, because {{ katex(body="i" )}} is prime and therefore would not be part of any of the previous iterations.

And so we can start our iteration from {{ katex(body="i^2" )}} instead of {{ katex(body="2i" )}}.

```python,linenos
N = 1000  # as per the problem statement
smallestPrimeFactor = [i for i in range(N+1)]
for i in range(2, N):
    if smallestPrimeFactor[i] == i:  # prime!
        for j in range(i*i, N+1, i):
            smallestPrimeFactor[j] = i
```
