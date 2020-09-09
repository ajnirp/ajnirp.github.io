---
layout: post
title: Why Moore' Voting Algorithm works
published: true
mathjax: true
---
Moore's Voting Algorithm finds the majority element in a homogenous array in a single-pass, with constant space. It's easy to see why that's optimal. The algorithm is based on a neat insight that's easy to derive and implement. This post explains that insight. The algorithm follows naturally. Without further ado:

A majority element occurs in more than half the indices in an array. So if an array has $n$ elements and $k$ of them are the majority elements, then $\frac{k}{n} \ge \frac{1}{2}$.
Iterate over the array. If two elements are unequal, remove them from the array. This is the insight: *the majority element won't change after removing those two unequal elements*. Why? Because by crossing out the two elements, you either decreased one instance of the majority element, or zero. Not two — because the elements are unequal.

If neither of the two crossed-out elements were the majority element the former majority element's new ratio in the new array is $\frac{k}{n-2}$. This is $\ge \frac{k}{n}$ (smaller denominator), which was already $\ge \frac{1}{2}$. And if we did remove a majority element instance, the new ratio is $\frac{k-1}{n-2}$. Is the new ratio greater than the old ratio, smaller than it, or equal to it, or does it depend on the values of $n$ and $k$? Derivation time:

$\frac{k-1}{n-2} \approx \frac{k}{n}\\
=> kn - n \approx kn - 2k\\
=> 2k \approx n\\
=> \frac{k}{n} \approx \frac{1}{2}$

The $\approx$ can be replaced by a $\ge$, and we're done. The new ratio is greater than the old one (regardless of what $k$ and $n$ were), and the old one was greater than $1/2$. So the former majority element is the new majority element. Neat.

The algorithm follows. We maintain a counter variable starting from 1 and decrement it when we see an element not equal to the current majority candidate. We increment it when we see the current majority candidate again. This generalises the idea of cancelling two elements. Here's some Python code for finding the majority element in an array guaranteed to have one.
```python
  count = 1
  majority_element = nums[0]
  for element in array[1:]:
      if element == majority_candidate:
          count += 1
      else:
          count -= 1
      if count == 0:
          majority_candidate = element
          count = 1
  return majority_candidate
```
In case the array contains elements that are expensive to copy around, you'd store the index of the candidate instead.
