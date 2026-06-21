+++
title = "Solving Project Euler #2 in J"
date = 2026-06-20
+++

In this post I discuss a fun problem I recently solved using [the J programming language](https://www.jsoftware.com/#/README). The problem is [number 2](https://projecteuler.net/problem=2) in Project Euler.

First, a quick disclaimer:

<div role="warning">
From the Project Euler <a href=https://projecteuler.net>FAQ</a>: "[...] the rule about sharing solutions outside of Project Euler does not apply to <b>the first one-hundred problems</b>, as long as any discussion clearly aims to instruct methods, not just provide answers, and does not directly threaten to undermine the enjoyment of solving later problems. Problems 1 to 100 provide a wealth of helpful introductory teaching material and if you are able to respect our requirements, then we give permission for those problems and their solutions to be discussed elsewhere."
</div>

## Problem statement

Paraphrasing, the problem asks us to

<div role="highlight">Find the sum of all even Fibonacci numbers that are under 4,000,000.</div>

The solution to this is straightforward in a programming language that supports the imperative paradigm. For example, in Python:

```python
def solve() -> int:
    prev, curr, result = 1, 2, 0
    while curr < 4_000_000:
        if curr % 2 == 0:
            result += curr
        prev, curr = curr, prev + curr
    return result

print(solve())
```

The result is `4613732`. How would we solve this problem in J?

## Solving this in J

Here's my solution. I'm still learning the language, so it's probably not very polished.

```j
step =. monad define
delta =. (curr , 0) {~ 2 | curr =. {: y
(delta + {. y) , curr , curr + 1 { y
)

{. (step^:(4000000 > {:))^:_ (0 1 2)
```

This is a straightforward implementation of the problem statement. Let's dissect it, starting with `step`.

`step` takes as input a **3-element array representing an accumulated value and two consecutive Fibonnaci numbers**. It returns **a new accumulated value and slides the two Fibonacci numbers forward**. The new accumulated value is the same as the old one if the old second Fibonacci number was odd. Otherwise, it's increased by the old second Fibonacci number.

Let's run this at the J REPL:

```j
   step 0 1 2
2 2 3
   step 2 2 3
2 3 5
   step 2 3 5
2 5 8
   step 2 5 8
10 8 13
```

The indented lines are our queries, and the non-indented lines are the result. We start with an accumulated value of `0`, and increase it only if we see an even Fibonacci number in the third position of the array.

The 2-line function translates this behavior directly. Starting with the first line:

* `y` represents the right-hand argument to `step`.
* `{:` takes the last element of an array.
* `curr =. {: y` assigns that value to `curr`.
* `2 | curr` computes the remainder of `curr` when divided by `2`.
* `(curr , 0)` is a 2-element array consisting of `curr` and `0`.
* `(2 | curr) { (curr , 0)` returns the `(2 | curr)`-th element of `(curr , 0)`.
* `~` is an adverb (function modifier) that flips its verb (function)'s operands (arguments). If `x` and `y` are two nouns (variables), then `x f~ y` is the same as `y f x`.
* And so `(curr , 0) {~ 2 | curr` is the same as `(2 | curr) { (curr , 0)`. I wrote it with the `~` because it reads more naturally to me.
* The above result is assigned to `delta`. This is what we'll add to the accumulated value.

Onto the next line.

* The result is composed of 3 elements joined by the `,` verb. They are: `delta + {. y`, and `curr`, and `curr + 1 { y`. Let's look at each one of them. 
* `{. y` is the first element of the 3-element array `y`. This is the accumulated value, to which we add `delta`, and that's the 1st element of the result.
* `curr` is the 2nd element of the result.
* `1 { y` is the 2nd element of the 3-element array `y`.
* `curr + 1 { y` is `curr` summed with that 2nd element. This becomes the 3rd element of the result.

That concludes our analysis of `step`. Nothing too fancy there. Onto the next line:

```j
{. (step^:(4000000 > {:))^:_ (0 1 2)
```

This line is an instance of a common J pattern `(u^:v)^:_ y`, where:
* `y` is a noun (variable).
* `u` is a monadic verb (single-argument function),
* `v` is a monadic boolean verb (single-argument function that returns either `0` or `1`), and

Here:

* `y` is (initially) the 3-element array `0 1 2` discussed earlier.
* `u` is the `step` function discussed earlier.
* `v` is `(4000000 > {:)`. It's a boolean verb that checks if the last element of an array (obtained by `{:`) is less than 4 million.

How does this pattern work?

### The while pattern

Let's discuss the pattern `(u^:v)^:_ y`. What's up with the two `^:`s and the underscore `_`?

`^:` is the **power** adverb. When coupled with a numeric noun it applies a function as many times as the noun specifies.

```j
    *: 3  NB. *: is the "square" verb.
9
    *: *: 3  NB. Square 3, and then square the result.
81
    (*:^:2) 3  NB. Same as the previous line. Square 3, and then square the result.
81
    (*:^:3) 3  NB. Apply the "square" verb thrice to 3.
6561
    (*:^:4) 3  NB. Apply the "square" verb four times to 3.
43046721
```

Side note: in J, `*:^:3 3` is not the same as `(*:^:3) 3`, which is why we added the parentheses. The former supplies an array `3 3` to the adverb `^:` and yields an unapplied verb. This behavior is not relevant here, so let's move on.

Let's give our boolean function `(4000000 > {:)` a nice name, `shouldContinue`. The final line of our J solution is then:

```j
{. (step^:shouldContinue)^:_ (0 1 2)
```

What does `step^:shouldContinue` do? `shouldContinue` is a verb and not a numeric noun, so the above explanation doesn't apply to it!

In plain language, for some argument `y`, the pattern `(step^:shouldContinue) y` is a verb that does the following: **apply `shouldContinue` to `y`. Take the result, which should be a numeric noun, and attach it to the `^:` that modifies `step`**.

This bears examining. `shouldContinue` is boolean, so it will yield either `0` or `1`. If `shouldContinue y` yields `1`, we apply `step` to `y`. If it yields `0`, we do nothing and just return `y`. In Python, this would be:

```python
def conditional_execution(u, v, y):
    return u(y) if v(y) else y
```

The fun part is when we attach a `^:_` adverb to this verb `step^:shouldContinue`. Recall that `_` in J is **positive infinity**. When `^:_` is attached to a verb, it means: **keep applying this verb until you reach a [fixed point of the function](https://en.wikipedia.org/wiki/Fixed_point_%28mathematics%29)**.

And so `(step^:shouldContinue)^:_` is a verb that runs `step^:shouldContinue` on its input until its output equals its input. And when does that happen? When `shouldContinue` yields `0`, and `y` is returned. At that point we've reached the fixed point of `step^:shouldContinue`, and we break out of the iteration!

In general, the J sentence `(u^:v)^:_ y` is interpreted as follows: repeatedly apply the verb `u` to the noun `y`, as long as the boolean verb `v` applied to the noun `y` returns `1` (true). This is the `while` loop we all know and love. Translating this to Python, this would be:

```python
def while_in_j(u, v, y):
    while True:
        if not v(y):
            return y
        y = u(y)
```

With all this in mind, we can translate the final line of the solution:

```j
{. (step^:(4000000 > {:))^:_ (0 1 2)
```

into plain language as follows:

<div role="highlight">Run the step function until the rightmost element of the input array is less than 4 million. Finally, return the leftmost element of the array (the accumulated value).</div>

## Doesn't J have a while. keyword?

J does have a `while.` keyword. We _could_ have written something like this.

```j
solve =. monad define
a =. 0 1 2
while. y > {: a do.
  a =. step a
end.
{. a
)
```

and we could've used some syntactic sugar and taken advantage of the way control structures are parsed to squeeze it into one line:

```j
solve =. {{ a =. 0 1 2 while. y > {: a do. a =. step a end. {. a }}
```

I haven't profiled this method compared with the previous one, but if I had to hazard a guess, I suspect this `while.` would be less efficient, because the J interpreter knows how to take advantage of common patterns like `(u^:v)^:_ y`. Plus, `while.` is considered unidiomatic in J anyway.

## Closing thoughts

There were some nice ideas in this otherwise straightforward implementation, but by far the most interesting one in my opinion is the idea to rephrase a `while` loop with a boolean breakout condition as an attempt to locate a fixed point of a function. This is part of the reason I love J, and languages like it. They encourage — or in some cases actively push — you to look at computational patterns in a wholly different way.

## References

* A great article by Jordan Scales on [Fibonacci numbers in J](https://thatjdanisso.cool/j-fibonacci). I adapted the idea in this post to incorporate an accumulated value.
* The excellent [J for C Programmers](https://www.jsoftware.com/help/jforc/contents.htm) book by Henry Rich. I can't recommend this enough.