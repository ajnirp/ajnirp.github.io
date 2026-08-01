+++
title = "Run-length encoding in APL"
date = 2026-07-20
+++

Run-length encoding (RLE) is a classic problem in information theory. This post discusses an APL solution to a common variant of the problem: given a numeric array, return a list of its runs. A run is a contiguous subarray with all elements equal, and so to represent a run we need two values: the value of each element and the length of the subarray.

For example, given the array `1 1 2 1 1 1 3 3 4 5 1` the output would be `(1 2) (2 1) (1 3) (3 2) (4 1) (5 1) (1 1)`. The first element of each pair is the value of the run. The second is the length of the run.

A function to implement RLE in Python might look like this:

```python
def rle(ls: list[int]) -> list[list[int]]:
    '''Returns a list of 2-element lists representing values and their run lengths'''
    result = []
    for val in ls:
        if result and val == result[-1][0]:
            result[-1][1] += 1
        else:
            result.append([val, 1])
    return result
```

Here's a function to do the same in APL. It accepts scalar and vector (rank 1 array) arguments and returns a table in which the first row is the run values and the second row is the run lengths.

```apl
{lens←2-⍨/⍸1,⍨mask←1,2≠/vec←,⍵ ⋄ ↑(mask/vec) lens}
```

We can run it:
```apl
      {lens←2-⍨/⍸1,⍨mask←1,2≠/vec←,⍵ ⋄ ↑(mask/vec) lens} 1 1 2 1 1 1 3 3 4 5 1
1 2 1 3 4 5 1
2 1 3 2 1 1 1
```

I like how this approach embodies the array paradigm: think in terms of array transformations instead of explicitly specifying iteration bounds. The resulting solution is almost declarative.

To start with we vectorize `⍵` in case it's a scalar. `2≠/vec` is a Boolean mask of when an element differs from the one to its right. Preprend a `1` to that and we have a mask telling us where each run starts, stored in `mask`. Append a `1` and use `⍸` to get the index of each `1`. These are the starting indices of each run. Find the run lengths by taking the pairwise differences, and then mix `↑` the values and the lengths to return our desired table.

Note the use of `¯2` instead of `2` when computing pairwise differences. Negative values reverse the arguments, and since the magnitude is 2, this amounts to a flip. We could've also written `2-⍨/`. Note also how we appended a `1`. That's a sentinel value that allows us to obtain the length of the final run.

A generalized solution to this problem can be found in `dfns.packR`. Peruse at your leisure. It also handles run-length expansion, and returns rich information about its result!

```apl
      'packR' ⎕cy 'dfns'
      packR
packR←{                             ⍝ Run Length Encoding (RLE packing).

    cmp←{                           ⍝ compress:
        shape←⍴⍵ ⋄ vect←,⍵          ⍝ shape and items of array.
        runs←1,2≠/vect              ⍝ mask of start of runs.
        repls←¯2-/⍸runs,1           ⍝ replication count.
        solos←runs/vect             ⍝ non-repeated items.
        shape repls solos           ⍝ compressed structure.
    }

    exp←{                           ⍝ expand:
        shape repls solos←⍵         ⍝ compressed structure.
        shape⍴repls/solos           ⍝ reconstituted array.
    }

    ⍺←1 ⋄ ⍺:cmp ⍵ ⋄ exp ⍵           ⍝ compress or expand.
}
```