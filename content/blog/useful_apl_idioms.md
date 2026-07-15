+++
title = "Useful APL snippets"
date = 2026-07-13
+++

## Editors

In RIDE, to display function trains in tree format:
```apl
    ]box ON -trains=tree
Was OFF -trains=box
    +/÷≢
  ┌─┼─┐
  / ÷ ≢
┌─┘
+
```

## Basic array operations

Turn a scalar into an array.
```apl
   ,5
5
   ⍴,5
1
```

First element of an array.
```apl
⊃
```

Last element of an array. Note the lack of symmetry with `⊃`.
```apl
⊢/
```

Flatten an array.
```apl
{,⍵}
```

Check that an array is a palindrome.
```apl
⌽≡⊢
```

Pad a number with a desired number of `0`s.
```apl
    11↑1 ⍝ 1 followed by ten 0's.
1 0 0 0 0 0 0 0 0 0 0
```

Find the number of leading `1`s in a boolean array.
```apl
    +/∧\
```

Find the corner elements of a table.
```apl
⊃     ⍝ top left
⊃⌽    ⍝ top right
⊃⊖    ⍝ bottom left
⊃⌽⍤⊖  ⍝ bottom right
```

Two ways to sum a boolean array.
```apl
+/
≢⍸
```

Empty out an array.
```apl
⍬⍨  ⍝ Using a constant function
0/  ⍝ Using replicate
```

Zero out selected indices from an array.
```apl
    a←⍳ 4
1 2 3 4
    a[2 3]←0
    a
1 0 0 4
```

Mean of a numeric array. Note that we do a `1⌈` on the `≢`. This ensures that for `0⍴0` the denominator is `1` rather than `0`, and so we evalute `0÷1` rather than `0÷0` which APL would compute as `1`.
```apl
+⌿÷1⌈≢
```

Find the first index of occurrence for each element of an array.
```apl
    ⍳⍨ 2 3 3 1 5 2
1 2 2 4 5 1
```

## Numbers

Sign of a number.
```apl
×
```

Get the integral and fractional part of a non-negative real number.
```apl
0 1∘⊤
```

## Profiling

Compare the performance of 3 functions `F`, `G` and `H` on the same input `s`.
```apl
'cmpx' ⎕CY 'dfns'
cmpx 'F s' 'G s' 'H s'
```

## Sorting

Sort an array in ascending order.
```apl
{⍵[⍋⍵]}
```

Order the rows of a matrix according to the sorted order of the second row.
```apl
    ⊢ mat ← 2 3 ⍴ 1 4 2 5 9 7
1 4 2
5 9 7
    mat[; ⍋mat[2;]]
1 2 4
5 7 9
```

Find the index of the maximum element of an array. Recall that in APL indices start at `1`.
```apl
⊃⍒
```

## Strings

All uppercase English letters.
```apl
    ⎕A
ABCDEFGHIJKLMNOPQRSTUVWXYZ
```

All lowercase English letters.
```apl
    ⎕UCS 96+⍳26
abcdefghijklmnopqrstuvwxyz
```

Discard all characters except English letters.
```apl
    a ← 'The quick brown fox! Jumps over the lazy... dog?'
    a∩⎕A,⎕UCS 96+⍳26
ThequickbrownfoxJumpsoverthelazydog
```

Convert a string to lowercase.
```apl
    ⎕C 'dYALog`
dyalog
    ¯1 ⎕C 'dYALog`
dyalog
```

Convert a string to uppercase.
```apl
    1 ⎕C 'dYALog`
dyalog
```

Case-insensitive string comparison. Note the use of `⍥` (over), which monadically applies its right function to both arguments, and then dyadically applies its left function to the results.
```apl
    'Dyalog' ≡⍥⎕C 'DYALOG'
1
```

Check if `⍺` is a prefix of `⍵` (works for numeric arrays too).
```apl
⊃⍷
```
For suffix, we use the same idea.
```apl
{⊃(⌽⍺)⍷⌽⍵}
```
 
Check that a string is a palindrome after discarding non-alphabet characters. We convert to uppercase, discard everything that is not an uppercase letter, and then check that the result is a palindrome.
```apl
{(⌽≡⊢) ⎕A∩⍨1⎕C ⍵}
```

## Mathematics

Generate an identity matrix.
```apl
∘.=⍨ ⍳
```
This is basically a tacit rephrasing of
```apl
{(⍳⍵)∘.=⍳⍵}
```

The following method is about as fast as it gets for generating an identity matrix. Right-pad `1` with `⍵` zeroes and then repeat that array until the shape is `⍵ ⍵`.
```apl
{⍵ ⍵⍴(⍵+1)↑1}
```

This can be rephrased in tacit form as a 5-train. Note that we use `×` (sign) to replace the positive input with `1`. We could also use `1⍨` or `≢` instead of `×`.
```apl
,⍨⍴+∘1↑×
```

Generate a random float between 0 and 1.
```apl
?0
```
