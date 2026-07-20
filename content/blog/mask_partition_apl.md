+++
title = "Partition arrays with a mask in APL"
date = 2026-07-19
+++

A pattern I've encountered a couple of times when solving [APL quest](https://apl.quest/) problems is we want to partition an array into two arrays based on its elements responding to a Boolean function. For example, in [2016.8](https://apl.quest/2016/8/)  we want to split a numeric array into two arrays. The first should contain negative elements, the second non-negative.

```apl
      (your_function) 0 1 ¯2 3 ¯4 ¯5 6 7 8 ¯9 10
┌───────────┬──────────────┐
│¯2 ¯4 ¯5 ¯9│0 1 3 6 7 8 10│
└───────────┴──────────────┘
```

We can obtain a Boolean mask with a `>` comparison.
```apl
      a ← 0 1 ¯2 3 ¯4 ¯5 6 7 8 ¯9 10
      mask ← 0>a
0 0 1 0 1 1 0 0 0 1 0
```

A nice idiom to join the mask with its negation is `,⍥⊂∘~⍨`. In this case we'd use it on `mask`:
```apl
      (,⍥⊂∘~⍨)mask
┌─────────────────────┬─────────────────────┐
│0 0 1 0 1 1 0 0 0 1 0│1 1 0 1 0 0 1 1 1 0 1│
└─────────────────────┴─────────────────────┘
```

I had trouble parsing this complex-looking function at first. Let's walk through it.
```apl
      (⊂mask),(⊂mask)
┌─────────────────────┬─────────────────────┐
│0 0 1 0 1 1 0 0 0 1 0│0 0 1 0 1 1 0 0 0 1 0│
└─────────────────────┴─────────────────────┘
      mask,⍥⊂mask
┌─────────────────────┬─────────────────────┐
│0 0 1 0 1 1 0 0 0 1 0│0 0 1 0 1 1 0 0 0 1 0│
└─────────────────────┴─────────────────────┘
      ,⍥⊂⍨mask
┌─────────────────────┬─────────────────────┐
│0 0 1 0 1 1 0 0 0 1 0│0 0 1 0 1 1 0 0 0 1 0│
└─────────────────────┴─────────────────────┘
```

We want to negate just the second mask, and then we can use each mask to our original array to get the final answer. Our operator of choice here is `∘`. I know that `X f∘g Y` expands to `X f g Y`. Let's call the `,⍥⊂` function `joinOverEnclose`.

```apl
      joinOverEnclose ← ,⍥⊂
```

Dyadic `joinOverEnclose` joins its arguments after enclosing each one of them. What we want in this case is to join two arguments after enclosing the left one and enclosing the _negation_ of the right one. In other words we want: 
```apl
      mask joinOverEnclose ~ mask
┌─────────────────────┬─────────────────────┐
│0 0 1 0 1 1 0 0 0 1 0│1 1 0 1 0 0 1 1 1 0 1│
└─────────────────────┴─────────────────────┘
```
which is just `mask joinOverEnclose∘~ mask` which is `joinOverEnclose∘~⍨ mask` which is `,⍥⊂∘~⍨`.

With the hard part behind us, we have our solution. We obtain our mask and negated mask, and apply each one of them to `⍵`. To do so we create a monadic function `/∘⍵` that has `⍵` bound to its right argument, thus leaving space for only a left argument, which is each one of our two masks.
```apl
      {/∘⍵¨(,⍥⊂∘~⍨)0>⍵} a
┌───────────┬──────────────┐
│¯2 ¯4 ¯5 ¯9│0 1 3 6 7 8 10│
└───────────┴──────────────┘
```