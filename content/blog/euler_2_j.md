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
    while curr < 4e6:
        if curr % 2 == 0:
            result += curr
        prev, curr = curr, prev + curr
    return result

print(solve())
```

The result is `4613732`. How would we solve this problem in J?

## A direct solution

Here's a direct solution. I'm still learning the language, so it's probably not very polished.

```j
step =. monad define
delta =. (curr , 0) {~ 2 | curr =. {: y
(delta + {. y) , curr , curr + 1 { y
)

{. (step^:(4e6 > {:))^:_ (0 1 2)
```

This is a straightforward translation of the problem statement into code. Let's dissect it, starting with `step`.

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
{. (step^:(4e6 > {:))^:_ (0 1 2)
```

This line is an instance of a common J pattern `(u^:v)^:_ y`, where:
* `y` is a noun (variable).
* `u` is a monadic verb (single-argument function),
* `v` is a monadic boolean verb (single-argument function that returns either `0` or `1`), and

Here:

* `y` is (initially) the 3-element array `0 1 2` discussed earlier.
* `u` is the `step` function discussed earlier.
* `v` is `(4e6 > {:)`. It's a boolean verb that checks if the last element of an array (obtained by `{:`) is less than 4 million.

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

Let's give our boolean function `(4e6 > {:)` a nice name, `shouldContinue`. The final line of our J solution is then:

```j
{. (step^:shouldContinue)^:_ (0 1 2)
```

What does `step^:shouldContinue` do? `shouldContinue` is a verb and not a numeric noun, so the above explanation doesn't apply to it!

In plain language, for some argument `y`, the pattern `(step^:shouldContinue) y` evaluates as follows: **apply `shouldContinue` to `y`. Take the result, which should be a numeric noun, and attach it to the `^:` that modifies `step`**.

This bears examining. `shouldContinue` is boolean, so it will yield either `0` or `1`. If `shouldContinue y` yields `1`, we apply `step` to `y`. If it yields `0`, we do nothing and just return `y`. In Python, this would be:

```python
def conditional_execution(u, v, y):
    return u(y) if v(y) else y
```

The fun part is when we attach a `^:_` adverb to this verb `step^:shouldContinue`. In J, `_` is **positive infinity**. When `^:_` is attached to a verb, it means: **keep applying this verb until you reach a [fixed point of the function](https://en.wikipedia.org/wiki/Fixed_point_%28mathematics%29)**.

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
{. (step^:(4e6 > {:))^:_ (0 1 2)
```

into plain language as follows:

<div role="highlight">Run the <code>step</code> function until the rightmost element of the input array (the "next" Fibonacci number) is <b>no longer</b> less than 4 million. Finally, return the leftmost element of the array (the accumulated value).</div>

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

I haven't profiled this method compared with the previous one, but if I had to hazard a guess, I suspect this `while.` would be less efficient, because the J interpreter knows how to optimize common patterns like `(u^:v)^:_ y`. Plus, `while.` is considered unidiomatic in J anyway.

## Beating the odds

The method above considers each Fibonacci number, asks if it is even, and adds it if yes. It would be nice to iterate in such a way that we skip all the odd numbers entirely. Does there exist a recurrence relation for the even Fibonacci numbers? I searched around and sure enough:

{% katex(block=true) %}
E_n = 4E_{n-1} + E_{n-2}
{% end %}

Here, {{ katex(body="E_n") }} is the n-th **even** Fibonacci number, with {{ katex(body="E_1 = 0") }} and {{ katex(body="E_2 = 2" )}}. This allows us to rewrite our `step` verb...

```j
step =. ({. + {:) , ({: , ({.@}. + 4*{:))
```

...and our `solve` verb...

```j
{. (step^:(4e6 > {.))^:_ (0 0 2)
```

...with the added bonus that `step` is now a tacit verb, meaning it makes no explicit reference to its operand!

The key to understanding `step` is to recognize that it's a fork, which is a way to compose three verbs. Suppose `f`, `g` and `h` are 3 monadic verbs, and `y` is a noun. The following two lines of J code are exactly identical.

```j
(f g h) y
(f y) g (h y)
```

In plain language:

<div role="highlight">Separately apply <code>f</code> to <code>y</code> and <code>h</code> to <code>y</code>. Use both values as operands to <code>g</code>.</div>

In the code above:

* `f` is `{. + {:` (itself a fork: add the first and last elements of an array).
* `g` is `,` (join two values into an array).
* `h` is itself a fork.
  * The first two verbs are `{:` (return the last element of an array) and `,` (join two nouns into an array).
  * The last one is `({.@}. + 4*{:)`, another fork (we're three levels deep now!).
    * The first verb is `{.@}.` (return the second element of an array).
    * The second is `+` (add two elements).
    * The third is `4*{:` (multiply the last element by 4).

We're doing essentially the same thing as in the first method:

* unconditionally add the old "second Fibonacci number" to the accumulated value.
* copy over the old "second Fibonacci number" to the new "first Fibonacci number".
* compute {{ katex(body="E_n = 4E_{n-1} + E_{n-2}") }} and copy it over to the new "second Fibonacci number".

Our starter value is now the array `0 0 2`.

The difference from the first method is now there's no check on the even-ness of the second Fibonacci number. It's always even!

## Keeping less state

Do we really need to store an accumulator value at every step of the iteration?

While searching around, I stumbled across this [article](https://www.geeksforgeeks.org/dsa/nth-even-fibonacci-number/) which demonstrates how to write the **sum** of the first {{ katex(body="N") }} Fibonacci numbers in terms of a linear combination of a constant number of Fibonacci numbers:

{% katex(block=true) %}
\sum_{i=1}^{n} F_i = F_{n+2} - 1
{% end %}

My first thought was if we could do the same for the sum of the first {{ katex(body="N") }} even Fibonacci numbers as well. I grabbed a pen and paper, and was able to derive:

{% katex(block=true) %}
\sum_{i=1}^{n} E_i = \frac{E_{n+1} + E_n - 2}{4}
{% end %}

(If you're interested in the derivation, it's in the [appendix](#appendix).)

This permits us to write our solve logic another way. Say {{ katex(body="k") }} is the last even Fibonacci number to be under 4 million. Then the sum of this number and all smaller even Fibonacci numbers is

{% katex(block=true) %}
\frac{E_{k+1} + E_k - 2}{4}
{% end %}

and so, as before, we stop our iteration once the second Fibonacci number is not less than 4 million:

```j
(step^:(4e6 > {:))^:_ (0 2)
```

Note that this time our starter value is just `0 2` — we've gotten rid of the accumulated value!

The output array contains two numbers, the first being the last even Fibonacci number under 4 million, and the second being the one just after it. We apply the formula above: sum them up, subtract `2`, and divide by `4`. In J, this would be:

```
x: 4 %~ _2 + +/ (step^:(4e6 > {:))^:_ (0 2)
```

The `x:` at the beginning forces higher precision. Without it, the result renders as `4.61373e6` instead of `4613732`.

Putting it all together, our third solution, and the shortest so far, is:

```j
step =. {: , ({. + 4*{:)
x: 4 %~ _2 + +/ (step^:(4e6 > {:))^:_ (0 2)
```

Notice how the `step` definition is much shorter now. We only need to advance the two Fibonacci numbers. No bothering with the accumulated value.

We can go one step further and get rid of the parentheses in the definition of `step`. As it happens, the [**train**](https://www.jsoftware.com/help/learning/09.htm) of five verbs in a row `(a b c d e f)` (when applied monadically i.e. with only a right argument) is evaluated like so:

```j
(a b c d e) y
(a y) b ((c d e) y)
(a y) b ((c y) d (e y))
```

In other words, a train of 5 verbs resolves into a fork whose right argument is a nested fork.

This is an instance of a more general rule, which is when you have an odd-length train of verbs `(a b c ...)` it is equivalent to a fork with the first verb being monadic `a`, the second verb being dyadic (i.e. with a left and right argument) `b`, and the third verb being monadic `(c ...)`. The last one is _another_ monadic odd-length train of verbs, and we can expand this recursively, as we did above for the train of 5 verbs.

Here, `a` is `{:` (take the rightmost element of an array), `b` is dyadic `,` (combine two elements into an array), `c` is `{.` (take the leftmost element of an array), `d` is dyadic `+` (sum two numbers), and `e` is `4*{:` (multiply the rightmost element of an array by 4).

And so we can discard the parentheses and just write

```j
step =. {: , {. + 4*{:
x: 4 %~ _2 + +/ (step^:(4e6 > {:))^:_ (0 2)
```

We can also get rid of one set of parentheses in the second line, turning `(u^:(v))^:_` into just `u^:(v)^:_`:

```j
step =. {: , {. + 4*{:
x: 4 %~ _2 + +/ step^:(4e6 > {:)^:_ (0 2)
```

In the interest of code golfing, we can combine the two lines and get rid of as many spaces as possible. We'd end up with:

```j
x:4%~_2++/({:,{.+4*{:)^:(4e6>{:)^:_(0 2)
```

There are likely better ways to solve this problem. Maybe I'll revisit this article in the future, when I know more J. For now, this is a good place to stop!

## Closing thoughts

There were some very cool ideas at play here. To list a few:
* iteration recast as a fixed point search
* iteration without explicit iteration variables
* tacit verb definitions and nested forks
* a recurrence for even Fibonacci numbers, and a summation formula for them

It's hard to pick a winner, but IMO the most interesting idea was that we can rephrase a `while` loop with a boolean breakout condition as a search for a fixed point of a function. This is part of the reason I love J, and languages like it. They encourage, or in some cases actively push you to look at computational patterns in a wholly different way.

## References

* A great article by Jordan Scales on [Fibonacci numbers in J](https://thatjdanisso.cool/j-fibonacci). I adapted the method in the post to incorporate an accumulated value.
* The excellent [J for C Programmers](https://www.jsoftware.com/help/jforc/contents.htm) book by Henry Rich. I can't recommend it enough. I read it to understand the `(u^:v)^:_` pattern.
* [Verb trains in J](https://www.jsoftware.com/help/learning/09.htm).
* GeeksForGeeks
  * [Summing the first N Fibonacci numbers](https://www.geeksforgeeks.org/dsa/nth-even-fibonacci-number/)
  * [A recurrence relation for even Fibonacci numbers](https://www.geeksforgeeks.org/dsa/sum-fibonacci-numbers/)

## Appendix

To derive the identity for the sum of the first {{ katex(body="N") }} even Fibonacci numbers, we start from:

{% katex(block=true) %}
E_n = 4E_{n-1} + E_{n-2} \\
\implies E_n - E_{n-1} = 3E_{n-1} + E_{n-2}
{% end %}

And plug in different values for {{ katex(body="n") }}, from {{ katex(body="3") }} up to {{ katex(body="n") }}:

{% katex(block=true) %}
E_n - E_{n-1} = 3E_{n-1} + E_{n-2} \\
E_{n-1} - E_{n-2} = 3E_{n-2} + E_{n-3} \\
... \\
E_3 - E_2 = 3E_2 + E_1 \\
{% end %}

Adding these all up, we get:
{% katex(block=true) %}
E_n - E_2 = 3\sum_{i=2}^{n-1} E_i + \sum_{i=1}^{n-2} E_i
{% end %}

Adjusting the indices a bit:

{% katex(block=true) %}
E_n - E_2 = 3\sum_{i=1}^{n-1} E_i + \sum_{i=1}^{n-1} E_i - 3E_1 - E_{n-1} \\
\implies E_n - E_2 = 4\sum_{i=1}^{n-1} E_i - 3E_1 - E_{n-1} \\
{% end %}

And now rearranging:

{% katex(block=true) %}
4\sum_{i=1}^{n-1} E_i = E_n + E_{n-1} - E_2 + 3E_1
{% end %}

Recall that {{ katex(body="E_1 = 0") }} and {{ katex(body="E_2 = 2") }}. Substituting these in, and replacing {{ katex(body="n") }} by {{ katex(body="n + 1") }} in the final expression, we get:

{% katex(block=true) %}
4\sum_{i=1}^{n} E_i = E_{n+1} + E_n - 2 \\
\implies \sum_{i=1}^{n} E_i = \frac{E_{n+1} + E_n - 2}{4}
{% end %}

and we're done!