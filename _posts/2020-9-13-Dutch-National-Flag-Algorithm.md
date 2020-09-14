---
layout: post
title: The insight behind the Dutch National Flag Algorithm
published: false
---
The Dutch National Flag Algorithm takes as input an array of comparable elements and a pivot. It outputs nothing, and after it's done running the array has its elements moved around in-place so that all elements less than the pivot come first, then all elements equal to the pivot, and then all elements greater than the pivot. This algorithm shows up in quicksort, of which it's the only really complicated part in my opinion. What makes it special is that it runs in one pass and uses only constant space. The intuition behind this algorithm and the way its indices change eluded me until recently, so I'm writing this post to explain it.

The idea is to maintain four sections in the array from left to right. Four sections means we need three indices to carve up the array - call them $i$, $j$ and $k$. The sections are:

1. Stuff less than the pivot i.e. indices $< i$
2. Stuff equal to the pivot i.e. indices $>= i$ and $< j$
3. Unclassified stuff i.e. indices $>= j$ and $< k$
4. Stuff greater than the pivot i.e. indices $>= k$

The key point is that these invariants are maintained throughout the algorithm. And if that's true, then we know what our indices should be initially.

* $i = 0$  - initially, nothing is less than the pivot - we haven't seen anything.
* $j = 0$ because everything is unclassified.
* $k = len(array)$ - again, nothing is greater than the pivot because we haven't seen anything.

With this knowledge and the assurance that our algorithm gets only one pass to run, we can figure out how to write the algorithm:

```python
def dnf(array, pivot):
  i, j, k = 0, 0, len(array)
  while j < k:
    if array[j] < pivot:
      array[i], array[j] = array[j], array[i]
      i += 1
      j += 1
    elif array[j] == pivot:
      j += 1
    else:
      k -= 1
      array[j], array[k] = array[k], array[j]
```
Let's analyse these cases.

If we see a greater-than-pivot element at index $j$, we decrement $k$ and swap the two elements. Note that the element that was formerly located at the decremented value of $k$ was unclassified. So index $j$ now stores an unclassified element. And so we don't increment $j$. $i$ doesn't need to change.

If we see a smaller-than-pivot element at index $j$, we can swap it with the one at index $i$. We then increment $i$ to maintain the invariant in item 1 above. Also, the element formerly located at $i$ was equal to the pivot TODO

If we see an equal-to-pivot element at index $j$, we increment $j$ to maintain the invariant in item 2. $i$ and $k$ don't need to change.
