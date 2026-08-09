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

An optimized expression will be colored differently. Examples: `⊃⌽` or `⊢/` or `+/∧\`. I use the Nord theme, and they're colored orange.

Exit RIDE
```apl
⍝ Like all commands, these are case-insensitive
⎕OFF
)OFF
```

## Miscellaneous array operations

Turn a scalar into an array. Quite useful in situations like dealing with single-character strings, which APL treats as scalars.
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

Select multiple indices from an array.
```apl
    a←'abcd'
    a[3 1 4 2]
cadb
    3 1 4 2 ⌷⍤0 99⊢'abcd'  ⍝ 99 is a conventional value to use for array ranks greater than Dyalog's maximum of 15
cadb
```

Check that an array is a palindrome.
```apl
⌽≡⊢
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

Find the _endings_ of occurrences of array `⍺` in array `⍵`. Recall that if we want the _beginnings_ we can just use `⍷`.
```apl
{(1-≢⍺)⌽⍺⍷⍵}
```

Mode of an array.
```apl
{m←⍉{⍺,≢⍵}⌸,⍵ ⋄ m[1;]/⍨(⊢=⌈/)m[2;]}
{⊃{⍺/⍨⍵=⌈/⍵}/,⌿,∘≢⌸⍵}  ⍝ A more elegant formulation by abrudz
```

Standard deviation of an array.
```apl
{0.5*⍨(×/⍴⍵)÷⍨+/(×⍨⊢-+/÷≢),⍵}
```

Prefixes of an array
```apl
{(⍳≢⍵)↑¨⊂⍵}
,\  ⍝ very bad performance, like 3 orders of magnitude worse
```

Check if a table is a magic square.
```apl
∧/⊢=⊃(+⌿),(+/),+/,⍥(1 1∘⍉)∘⌽
(∧/⊢=⊃)(+⌿),(+/),(+⌿1 1∘⍉∘⌽),(+⌿1 1∘⍉)  ⍝ Identical to the above, but more verbose
```

Run-length encoding (RLE) / run-length packing.
```
'packR'⎕cy'dfns'
packR  ⍝ To study the source code
⍝ See also {⌈/¯2-/⍸1,⍨1,2≠/×2-/⍵} which is the solution to https://apl.quest/2021/9/
⍝ The high-level idea is: pairwise differences, signum, pairwise inequality, pad left and right with 1,
⍝ find indices of 1s, then find the consecutive differences. These are the starts of the runs.
⍝ packR does some extra stuff which I haven't looked at yet.
```

### Boolean arrays

Convert a positive number to binary by using the inverse of the decode function.
```apl
2⊥⍣¯1⊢
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

Generate a random array of `0`s and `1`s with length `1000`.
```apl
    v←{⎕IO←0 ⋄ ?⍵⍴2}1000
```

Find the first `1` in a Boolean vector.
```apl
<\
```
At first glance, this looks like it would run in quadratic time, but APL interpreters optimize for this and the actual runtime is linear.

## Numbers

Sign of a number.
```apl
×
```

Get the integral and fractional part of a non-negative real number.
```apl
0 1∘⊤
```

Convert a positive number to its digits. See [this StackOverflow answer](https://stackoverflow.com/a/44990891).
```apl
10∘⊥⍣¯1
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

All digits from `0` to `9`.
```apl
    ⎕D
0123456789
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

Split a string `⍵` into a words on a delimiter `⍺`, discarding empty strings.
```apl
{+/2</1,⍨⍺=⍵}  ⍝ We're looking for "0 1" subarrays in the is-mask. We add a 1 at the end to ensure there's a "0 1" subarray for the last word.
```
Split a string `⍵` into a words on any one of character vector of delimiters `⍺` (length 1 or more), keeping empty strings.
```apl
{1↓¨x⊂⍨⍺∊⍨x←⍵,⍨⊃⍺}  ⍝ 2016.9. Split on a set of delimiters, keeping empty strings.
```
 
Check that a string is a palindrome after discarding non-alphabet characters. We convert to uppercase, discard everything that is not an uppercase letter, and then check that the result is a palindrome.
```apl
{(⌽≡⊢) ⎕A∩⍨1⎕C ⍵}
```

Compare semver strings, assuming they're represented as arrays of 3 numbers. Subtract, find the signs and return the first non-zero element (or just `0`).
```apl
{⊃d/⍨0≠d←×⍺-⍵}
```
Another nice way. This relies on `⊃⍬` being `0`. Note also the use of `0⍨` as the right tine of the fork.
```apl
⊃⍤×-~0⍨
```
A very clever way to do this is as follows. We decode the signum'd array of `1`s, `0`s and `¯1`s in base 2. If the result is negative it means the highest non-zero value must have been `¯1` and likewise if the result is positive then the highest non-zero value must have been `1`.
```apl
×2⊥×⍤-
```

Join a vector of words into a sentence, separated by spaces.
```apl
      ⊃(⊣,' ',⊢)/ 'this' 'is' 'a' 'vector' 'of' 'words'
this is a vector of words
```

Reverse individual words within a sentence while preserving the word order. The result is always a vector.
```apl
{⍵≡'': ⍵ ⋄ ⊃(⊣,' ',⊢)/⌽¨(' '∘≠⊆⊢),⍵}
      {⍵≡'': ⍵ ⋄ ⊃(⊣,' ',⊢)/⌽¨(' '∘≠⊆⊢),⍵}'this is a sentence'
siht si a ecnetnes
```

Convert an Excel column identifier to a number.
```apl
{26⊥⎕A⍳⍵}
26⊥⎕A⍳⊢  ⍝ Tacit version of the above
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

Set the random seed.
```apl
⎕RL
```

Pascal's triangle
```apl
{{(⍵-1)!⍨¯1+⍳¨⍵}⍳⍵}
(∘.!⍨0,⍳)  ⍝ From https://aplcart.info
```

## Time

Get the current time.
```apl
⎕TS  ⍝  Returns a 7-element integer vector representing the current year, month, day, hour, minute, second, and millisecond in that order. 
```

Compare two timestamps that are both in the format returned by `⎕TS`. Return `¯1` if the left timestamp is smaller, `0` if they're equal, and `1` if the left timestamp is greater.
```apl
×2⊥×⍤-  ⍝ This is identical to the semver comparison algorithm!
```

Check if a year is a leap year
```apl
((0=4∘|)∧(0≠100∘|))∨0=400∘|  ⍝ My first attempt - clumsy, in retrospect
≠⌿0=400 100 4∘.|⊢  ⍝ Key idea here is we have a yes-no-yes type of situation which is perfect for xor i.e. ≠
```