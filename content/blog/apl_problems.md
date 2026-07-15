+++
title = "Solutions to some APL problems"
date = 2026-07-14
+++

## Dyalog APL challenge

### 2026.2 - 9

Write a function that takes an uppercase word consisting of English letters and counts how many leading consonants it has. For the purposes of this problem, use 'AEIOU' as your list of vowels (non-consonants).

```apl
{+/∧\1-∨⌿'AEIOU'∘.=⍵}
```
We build a _table_ of vowel occurrences and do a logical OR along the columns to get an _array_ of vowel occurrences (`0` for consonants and `1` for vowels). Then we NOT the array and use `+/∧\` to count how many leading `1`s it has.

### 2026.2 - 10

Write a function that takes an uppercase word consisting of English letters and returns the first vowel followed by the first consonant. The word is guaranteed to contain at least one vowel and one consonant.

```apl
{⍵[1 0⍳⍨∨⌿'AEIOU'∘.=⍵]}
```
As before `∨⌿'AEIOU'∘.=⍵` generates an array of vowel occurrences. Call the ⌈result `v`. `v⍳1 0` gives us the indices of the first vowel and first consonant, and we pick those indices from `⍵`.

## APL quest problems

### 2018.1

[Problem](https://apl.quest/2018/1/).
```apl
{≢ ∪ ⌈\ ⍵}
{+/ ≠ ⌈\ ⍵}
```
Compute the max-scan, then ask how many unique elements there are.

Another nice idea is to compare pairs of adjacent numbers in the max-scan and add up the points of difference. We'll need add `1` to get the right answer, and also special-case the empty array case. 

```apl
{⍬≡⍵: 0 ⋄ 1+ +/ 2</ ⌈\ ⍵}
```

This is more efficient because there's no hashing involved. We're making use of the structure of the max-scan (strictly increasing). We can use `cmpx` to profile these implementations.

### 2018.2

[Problem](https://apl.quest/2018/2/)

The fractional part is the remainder modulo `1`, and we can subtract that from the input to get the integral part.
```apl
{(⍵-rem),rem←1|⍵}
```

We can use `⌊` to compute the integral part, and then subtract that from the input to get the fractional part.
```apl
{⌊ , ⍵-⌊⍵}
```
We can rewrite the right argument to `,` as a 3-train `⊢-⌊` and then go one step further and turn the entire thing into a 5-train.
```apl
⌊ , ⊢ - ⌊
```

Combining with the previous idea:
```apl
⌊ , 1∘|
```

Or just use the encode function:
```apl
0 1∘⊤
```

### 2018.3

[Problem](https://apl.quest/2018/3/)

This uses the idiom `⍺⍨¨⍵` which creates an array in the "shape" of `⍵`, with each element being `⍺`. The actual shape of the result will be different if `⍺` is itself an array.
```apl
{{⍺ ('*'⍨¨⍵)} ⌸ , +/¨ ⍳⍵}
```

For example, try
```apl
(2 4 ⍴ ⍳ 8) ⍨¨ ⍳ 3
```

To avoid the problem of generating high-rank matrices (which caps us at inputs of length 15 or less), we can use encode:
```apl
{{⍺ ('*'⍨¨⍵)}⌸+⌿1+(,⍵)⊤¯1+⍳×/⍵}
```
Notice how we have to use `,⍵` instead of `⍵` as  the left arugment to `⊤` (encode). This handles the case of when `⍵` is a scalar.