+++
title = "Maximum nested paren depth in APL"
date = 2026-08-01
+++

Let's solve a [Leetcode problem](https://leetcode.com/problems/maximum-nesting-depth-of-the-parentheses/description/) in APL. This blog post is inspired by this [code report video](https://www.youtube.com/watch?v=zrOIQEN3Wkk). Given a valid parentheses string `s`, return the nesting depth of `s`. The nesting depth is the maximum number of nested parentheses.

My approach:

```apl
{⌈⌿+\¯1+2×'('=⍵∩'()'}
```

In plain language:
1. Discard all non-paren characters.
2. Create a mask of `(` characters.
3. Multiply by 2 and subtract 1 to get a mask of `1` and `¯1` instead of `1` and `0`.
4. Return the maximum prefix sum.

The idea in the last two steps is that we increment the depth when we see a `(`, decrement it when we see a `)`, and look for the maximum value we attain at any point.

I also quite like the trick in step 3 in which we convert a Boolean mask to a mask of `1` and `¯1`. [The video](https://www.youtube.com/watch?v=zrOIQEN3Wkk) mentions another a way to do it: the fork `(⊢-~)`, which simply subtracts from a mask its negation, yielding the same result. We can compare the runtimes of the two approaches.

```apl
      f←⊢-~
      g←{¯1+2×⍵}
      w←¯1+?10000⍴2
      'cmpx'⎕cy'dfns'
      cmpx 'f w' 'g w'
  f w → 1.0E¯6 |   0% ⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕
  g w → 8.8E¯7 | -14% ⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕     
```

which tells us that both approaches are identically fast within a margin of error.