+++
title = "Understanding the Dutch National Flag algorithm"
date = 2025-11-09
+++

In computer science, the [Dutch National Flag problem](https://en.wikipedia.org/wiki/Dutch_national_flag_problem) asks: can you take an array of numbers and a "pivot" value, and sort the array in-place so that:

* all elements less than the pivot value come first, followed by
* all elements equal to the pivot value, followed by
* all elements greater than the pivot value?

This problem shows up in [quicksort](https://en.wikipedia.org/wiki/Quicksort), of which it’s the only really complicated part in my opinion.

There's an elegant algorithm to solve this problem. It makes a single pass over the array and uses constant extra space! The implementation is easy enough to understand, but I had trouble understanding why it worked at all. That is, until I looked at it from the perspective of *maintaing invariants*.

The key idea is to **define a set of invariants and ensure that they are all true at the beginning of every loop iteration, and also after all iterations are completed**. In other words, we must ensure they're all true before any iterations have occurred, before the second iteration starts, before the third iteration starts... and finally when all iterations have completed.

What should our invariants look like? Well, they follow from the problem statement. Let's use three indices `i, j, k` to carve up our array into four sections, and choose our invariants to be:

1. Items less than the pivot are at indices `< i`
1. Items equal to the pivot are at indices `>= i` and `< j`
1. Unclassified items are at indices `>= j` and `< k`
1. Items greater than the pivot are at indices `>= k`

At the end of our loop iterations, no element should remain unclassified, which means section 3 should be empty, which means our terminating condition is `j == k`.

Knowing that we want our invariants . Before the iteration starts, assuming the array is 0-indexed, the indices should be initialized as follows:

* i = 0. Initially, nothing is less than the pivot - we haven’t seen anything.
* j = 0. everything in the range [0, len(array)) is unclassified.
* k = len(array). Nothing is greater than the pivot because we haven’t seen anything.

The hard part is to figure out what goes in the loop body. Remember, whatever we do in a single loop iteration, we must end with all invariants remaining true. At the same time, we want to make progress. In our formulation, progress means either incrementing `j` or decrementing `i`.

In this article we'll skip over how to derive the algorithm from first principles, and instead directly present it. Here it is, in Rust:

```rust,linenos
fn dnf<T: PartialOrd>(array: &mut [T], pivot: T) {
    if array.is_empty() {
        return;  // Nothing to do here.
    }

    let mut i = 0usize;
    let mut j = 0usize;

    // We checked for an empty array earlier, so this statement is guaranteed to
    // not underflow the `usize` type.
    let mut k: usize = array.len() - 1;

    while j < k {
        if array[j] < pivot {
            array.swap(i, j);
            i += 1;
            j += 1;
        } else if array[j] == pivot {
            j += 1;
        } else {  // array[j] > pivot
            k -= 1;
            array.swap(j, k);
        }
    }
}
```

The if conditions that make up the body of the loop are clear. But how do we know which indices to update inside the if conditions? The underlying idea is: any time it's safe to "advance" an index (increment in the case of `i` and `j`, decrement in the case of `k`), do it.

Let's analyze this case by case!

### `array[j] < pivot`

After incrementing indices `i` and `j`, and swapping the elements at those incremented indices:

* We first can swap the elements are indices `i` and `j`. Next, we increment index `i` because it's safe to do so - everything to the left of index `i` is still less than the pivot even after incrementing. This means invariant 1 is maintained!
* Next, notice how the element that used to be at index `i` was equal to the pivot (because of invariant 2), and now it's at index `j`. So we increment index `j` since invariant 2 holds even after incrementing `j`.
* Invariant 3 is still true, because we now have one less unclassified element.
* Invariant 4 is still true, because index `k` didn't change.

### `array[j] == pivot`

After incrementing `j`:

* Invariant 1 is still true, because index `i` didn't change.
* Invariant 2 is still true. All array elements between indices `i` and `j` (excluding the element at index `j`) are equal to the pivot.
* Invariant 3 is still true, because we now have one less unclassified element.
* Invariant 4 is still true, because index `k` didn't change.

### `array[j] > pivot`

We first decrement k and then swap the elements at indices j and k. This preserves invariant 4.

* Invariants 1 and 2 are still true, because neither index `i` nor index `j` changed.
* Invariant 3 is still true, because we now have one less unclassified element.
* Invariant 4 is still true. Why? Because initially the element at index `j` was greater than the pivot. We decremented index `k` and then swapped the elements at indices `j` and `k`, so now the element at index `k` is greater than the pivot. All elements to the right of the new value of index `k` continue to be greater than the pivot, as they were before. 

When the iteration ends, all invariants still hold. Additionally, the "unclassified" group empty (because `j == k`), and so all elements are sorted.

If you find this sort of invariant-based analysis interesting, consider reading up on [Program Derivation](https://en.wikipedia.org/wiki/Program_derivation), in particular the book Programming: The Derivation of Algorithms by Anne Kaldewaij. A powerful idea in program derivation is that you can write a program by methodically translating invariants into pseudocode. This is nice because it's typically much easier to formulate invariants for a given problem statement than to directly write a bug-free program.