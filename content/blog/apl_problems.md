+++
title = "Solutions to some APL problems"
date = 2026-07-14
+++

## Dyalog APL challenge

### 2026.2 - 9

Write a function that takes an uppercase word consisting of English letters and counts how many leading consonants it has. For the purposes of this problem, use 'AEIOU' as your list of vowels (non-consonants).

```apl
{+/∧\~∨⌿'AEIOU'∘.=⍵}
```
We build a _table_ of vowel occurrences and do a logical OR along the columns to get an _array_ of vowel occurrences (`0` for consonants and `1` for vowels). Then we NOT the array and use `+/∧\` to count how many leading `1`s it has.

### 2026.2 - 10

Write a function that takes an uppercase word consisting of English letters and returns the first vowel followed by the first consonant. The word is guaranteed to contain at least one vowel and one consonant.

```apl
{⍵[1 0⍳⍨∨⌿'AEIOU'∘.=⍵]}
```
As before `∨⌿'AEIOU'∘.=⍵` generates an array of vowel occurrences. Call the result `v`. `v⍳1 0` gives us the indices of the first vowel and first consonant, and we pick those indices from `⍵`.

## APL quest problems

### 2014
```apl
{(⊢≡⌽)⍵/⍨⍵∊⎕A,⎕UCS 96+⍳26}  ⍝ 2014.5
```

### 2015
```apl
(≡⍥{t[⍋t←s/⍨(s←1⎕C⍵)∊⎕A]})  ⍝ 2015.1
{0.5*⍨(×/⍴⍵)÷⍨+/(×⍨⊢-+/÷≢),⍵}  ⍝ 2015.5
```

### 2016.4

```apl
{h ← (≢ ⍺) ⌊ (≢ ⍵) ⋄ (h ↓ ⍺) ,⍨ (h ↓ ⍵) ,⍨ , ⍉ ↑ (h ↑ ⍺) (h ↑ ⍵)}  ⍝ My initial approach
```

### 2017.1

[Link to problem](https://apl.quest/2017/1/)
```apl
{¯1+2×⍳⍵}
{+\2-⍵↑1}  ⍝ doesn't rely on ⎕IO being 1!
```

### 2017.9

[Link to problem](https://apl.quest/2017/9/)
```apl
{2>≢⍵: (≢⍵)⍴0 ⋄ 0,⍨2=/⍵}
```

### 2017.10

[Link to problem](https://apl.quest/2017/10/)
```apl
{(s,s)⍴⍵↑⍨*⍨s←⌈0.5*⍨≢⍵}
```

### 2018.1

[Link to problem](https://apl.quest/2018/1/)
```apl
{≢ ∪ ⌈\ ⍵}
+/⍤≠⌈\
```
Compute the max-scan, then ask how many unique elements there are.

Another nice idea is to compare pairs of adjacent numbers in the max-scan and add up the points of difference. We'll need add `1` to get the right answer, and also special-case the empty array case. 

```apl
{⍬≡⍵: 0 ⋄ 1+ +/ 2</ ⌈\ ⍵}
```

This is more efficient because there's no hashing involved. We're making use of the structure of the max-scan (strictly increasing). We can use `cmpx` to profile these implementations.

### 2018.2

[Link to problem](https://apl.quest/2018/2/)

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

[Link to problem](https://apl.quest/2018/3/)

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
Notice how we have to use `,⍵` instead of `⍵` as  the left arugment to `⊤` (encode). This handles the case of when `⍵` is a scalar by promoting it to a vector.

### 2019
```apl
{⍵⊂⍨(⍺↑1)⍴⍨≢⍵}  ⍝ 2019.1
{'FDCBA'[0 65 70 80 90⍸⍵]}  ⍝ 2019.2
{m/⍨(∧/0∘<∧9∘>)¨m←(⊂⍵)+,(2 1)(1 2)∘.×∘.,⍨¯1 1}  ⍝ 2019.4. Knight's moves. Good use of outer product
```

### 2018.10

[Link to problem](https://apl.quest/2018/10/). Check if `⍺` and `⍵` are anagrams, ignoring spaces. Assume both are uppercase.

```apl
≡⍥{' '~⍨s[⍋s←,⍵]}
```

You could go with
```apl
≡⍥{' '~⍨⍵[⍋s⍵]}
```
but this runs into trouble with inputs like `'d'` which APL treats as scalars. Raveling and assigning to a variable guards against that.

### 2020.2

[Link to problem](https://apl.quest/2020/2/). It's a 5-train!
```apl
⊢⊂⍨128∘>∨191∘<
```

### 2020.5

[Link to problem](https://apl.quest/2020/5/). Note the conditional flip idiom. We subtract the right from the left, find the magnitude, index that many numbers, add the smaller number as an offset, and then conditionally flip the result if the left number is larger.
```apl
{⌽⍣(>/⍵)⊢(⌊/⍵)+0,⍳|-/⍵}
```

### 2020.6
```apl
{⍵[⍒⍺=⍵]}`
```

### 2022.5

[Link to problem](https://apl.quest/2022/5/). I had to assign the outer `⍵` to a new name so as to refer to it in the inner dfn. Otherwise, it is shadowed by the inner dfn's `⍵`. The core idea is that we overtake (with a negative amount) from an ever-increasing chain of `⎕`s. To see how this solution works, try running suffixes of it, like `{{x↑⍵⍴'⎕'}¨⍳x←⍵}7`.

```apl
{↑{(-x)↑⍵⍴'⎕'}¨⍳x←⍵}
```

Shortly after, I found a better way to rephrase the above idea. The nice thing here is that `↑` does the padding with prototypes while lifting the pre-result up one level of depth.
```apl
{⌽↑(⍳⍵)⍴¨'⎕'}
```
Similary:
```apl
{⌽↑'⎕'/¨⍨⍳⍵}
⌽⍤↑'⎕'/¨⍨⍳  ⍝ Tacit version of the above
```

### 2020.7
```apl
{b←2⊥⍣¯1⊢⍺ ⍵ ⋄ b[;1]≡∧/b}
(∧/≤/)2⊥⍣¯1,  ⍝ Much nicer - why use AND and compare against the original when you can just use ≤
```

### 2023.3

[Link to problem](https://apl.quest/2023/3/). Write a Caesar cipher. Use `' ',⎕A` as the alphabet.

```apl
      4 (your_function) 'HELLO WORLDS'
LIPPSD SVPHW

      ¯4 (your_function) 'HELLO WORLDS' ⍝ negative shifts are okay
DAHHKWSKNH O 

      0 (your_function) 'HELLO WORLDS' ⍝ no shift is okay
HELLO WORLDS

      27 (your_function) 'HELLO WORLDS'
HELLO WORLDS

      ¯10 (your_function) '' ⍝ returns an empty vector
```
My solution:
```apl
{a[⍵⍳⍨(-⍺)⌽a←' ',⎕A]}
```
Create the alphabet, search for `⍵` in the rotated alphabet, and then pick the resultant indices from the original alphabet.