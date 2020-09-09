---
layout: post
title: Why Moore' Voting Algorithm works
published: true
mathjax: true
---
Moore's Voting Algorithm finds the majority element in a homogenous array in a single-pass, with constant space. It's easy to see why that's optimal. The algorithm is based on a neat insight that's easy to derive and implement. This post explains that insight. The algorithm follows naturally. Without further ado:

A majority element occurs in more than half the indices in an array. So if an array has $n$ elements and $k$ of them are the majority elements, then $\frac{k}{n} \ge \frac{1}{2}$.
Iterate over the array. **If two elements are unequal, pretend the array doesn't have them**. This is the insight: *the majority element hasn't changed*. Why? Because by crossing out the two elements, you either decreased one instance of the majority element, or zero. Not two — because the elements are unequal.

If neither of the two crossed-out elements were the majority element the former majority element's new ratio in the new array is $\frac{k}{n-2}$. This is $\ge \frac{k}{n}$ (smaller denominator), which was already $\ge \frac{1}{2}$. And if we did remove a majority element instance, the new ratio is $\frac{k-1}{n-2}$. Is the new ratio greater than the old ratio, smaller than it, or equal to it, or does it depend on the values of $n$ and $k$? Derivation time:

$\frac{k-1}{n-2} \ge ? \frac{k}{n}\\\
=> kn - n \ge ? kn - 2k\\\
=> 2k \ge ? n\\\
=> \frac{k}{n} \ge ? \frac{1}{2}$

The $\geq$ can be replaced by a $\ge$, and we're done. The new ratio is greater than the old one (regardless of what $k$ and $n$ were), and the old one was greater than $1/2$. So the former majority element is the new majority element!
