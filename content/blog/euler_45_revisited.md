+++
title = "Revisiting Project Euler #45"
date = 2025-12-20
+++

In this blog post I revisit a fun Project Euler problem. The solution I detailed in this [previous blog post](/blog/euler-45) worked but was a little clunky, involving checking if a quantity was a perfect square. I submitted that blog post to Hacker News for discussion, and one of the [comment threads](https://news.ycombinator.com/item?id=45942720) on that submission opened up an interesting line of discussion — what if we just modeled this problem as a combination of iterators?

I translated the approach discussed in the above comments to Python, and got:

```python,linenos
# generators for pentagonal and hexagonal numbers
def pentagonal():
    i = 1
    while True:
        yield i*(3*i-1)//2
        i += 1
def hexagonal():
    i = 1
    while True:
        yield i*(2*i-1)
        i += 1

# given two generators, advances both until a common element is found, then
# yields that element
def combine(g1, g2):
    while True:
        l, r = next(g1), next(g2)
        while l != r:
            if l < r:
                l = next(g1)
            else:
                r = next(g2)
        yield l

if __name__ == '__main__':
    seen = 0
    for num in combine(pentagonal(), hexagonal()):
        print(num)
        seen += 1
        if seen == 4:  # change this to the number of desired outputs
            break
```

What's going on here? To start with, we're defining a function `combine` that takes as input two existing generators and only yields elements that are members of _both_ generators. This is ensured by the `while l != r` loop that precedes the `yield l`.

Fair enough, but how come the above code makes no mention of triangle numbers at all? Seems like we should define a generator for triangle numbers, and then do something like this?

```python
for num in combine(triangle(), combine(pentagonal(), hexagonal())):
    pass  # seen = 0 ...
```

It turns out that we don't need to! A nice insight from the [previous blog post](/blog/euler-45) that eluded me until I re-read the Hacker News thread (in particular, [this comment](https://news.ycombinator.com/item?id=45991571)) was that **every hexagonal number is a triangle number**. Recall that if the {{ katex(body="t^{th}") }} triangle number equals the {{ katex(body="h^{th}") }} hexagonal number, then {{ katex(body="t = 2h - 1") }} must be true.

We can read this property in reverse: given the {{ katex(body="h^{th}") }} hexagonal number, there always exists a corresponding {{ katex(body="t = 2h-1") }} such that the {{ katex(body="t^{th}") }} triangular number equals it. In other words, every hexagonal number is a triangle number. Which means that it's enough to check for numbers that are both hexagonal and pentagonal. The "triangle" requirement will automatically be satisfied 🙂.

This approach is much more readable than the one in the previous blog post, but is it more performant? As it turns out, this approach runs faster on my machine than the previous approach! It's not instantly obvious to me why, without inspecting the generated CPython bytecode, but I speculate that it has to do with the expensiveness of computing the floating-point square root of a number.