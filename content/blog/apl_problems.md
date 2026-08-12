+++
title = "Solutions to some APL problems"
date = 2026-07-14
+++

## Fun problems

```apl
{≠/¨2|⍳⍵ ⍵}  ⍝ checkerboard pattern
{⍵-⌈/¨|⍵-⍳,⍨(2×⍵)-×⍵}  ⍝ 2022.6. Pyramid elevations seen from above.
```

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

## APL quest

### 2013
```apl
{1=∧/~s←' '≠⍵: 0 ⋄ ≢s⊆⍵}  ⍝ 2013.3

⍝ 2013.4
⍝ Increment for opening, decrement for closing. The result should only contain
⍝ positive numbers and end in a 0.
{''≡⍵: 1 ⋄ ((0=⊢/)∧(∧/0∘≤))+\-⌿'()'∘.=⍵}

{⍵/⍨⍵≠⌊⍵}  ⍝ 2013.7

∘.×⍨⍳  ⍝ 2013.8

(+/÷≢)¨(⊣,/⊢)  ⍝ 2013.9

{(⌹⍵)+.×⍺}  ⍝ 2013.10
```

### 2014
```apl
⍝ 2014.1
{(×⍨⍵)=+.×⍨⍺}
{(⍵×⍵)=+/⍺×⍺}  ⍝ More readable IMO

⍝ 2014.2
⍝ Prepend and append a space and call this w. Then keep all characters that satisfy ANY of the following:
⍝ * not a vowel
⍝ * vowel and has a space to its left
⍝ * vowel and has a space to its right
⍝ Here, n is a mask of non-vowels. We OR its negation with a left and right shift of s, which is a mask
⍝ of spaces, and OR that with n. The resultant mask is then applied to w.
⍝ The use of ⌽¨⊂ was inspired by John Scholes' implementation of Conway's Game of Life.
{n←~'AEIOUaeiou'∊⍨w←' ',⍵,' ' ⋄ 1↓¯1↓w/⍨∨⌿n⍪↑~n∘∨¨(1 ¯1)⌽¨⊂' '≠w}

{⍵=0:⍬ ⋄ ({⍵,+/¯2↑⍵}⍣(⍵-1)),1}  ⍝ 2014.3

{(⊢≡⌽)⍵/⍨⍵∊⎕A,⎕UCS 96+⍳26}  ⍝ 2014.5

⍝ 2014.8
{⍵=|+/¯9 ¯11○⍺}  ⍝ Interpret the left argument as a complex number.
{(⍵×⍵)=+.×⍨⍺}  ⍝ Straightforward
{0.5*⍨+.×⍨⍺-⍵}  ⍝ Also straightforward

{9.8÷⍨(1○○⍵÷90)×⍺×⍺}  ⍝ 2014.9

{100×⌈⌿(¯1↓⍵)÷⍨1↓⍵-¯1⌽⍵}  ⍝ 2014.10
```

### 2015
```apl
⍝ 2015.1
(≡⍥{t[⍋t←s/⍨(s←1⎕C⍵)∊⎕A]})
≡⍥{t[⍋t←⎕A∩⍨1⎕C⍵]}
≡⍥{⊂⍤⍋⍛⌷⎕A∩⍨1⎕C⍵}  ⍝ Works on my machine (TM) but not in https://apl.quest

{0.5*⍨(×/⍴⍵)÷⍨+/(×⍨⊢-+/÷≢),⍵}  ⍝ 2015.5

⍝ 2015.8
{(¯0.01∘+)@(13∘=)⍵}
{⍵+¯0.01×13=⍵}

{1≥≢⍵: ⍵ ⋄ ∊⌽¨⍵⊂⍨1,1,⍨(¯2+≢⍵)⍴1 0}  ⍝ 2015.9
```

### 2016

```apl
+/÷1⌈≢  ⍝ 2016.1

⍝ 2016.4
{h ← (≢ ⍺) ⌊ (≢ ⍵) ⋄ (h ↓ ⍺) ,⍨ (h ↓ ⍵) ,⍨ , ⍉ ↑ (h ↑ ⍺) (h ↑ ⍵)}  ⍝ My initial approach, rather clumsy
{(⍺,⍵)[⍋(⍳≢⍺),⍳≢⍵]}  ⍝ Neat use of grade

{⊃(//,⌿{⍺,⍨1=≢⍵}⌸⍵)}  ⍝ 2016.5

{⍵[⍋≢¨⍵]}  ⍝ 2016.6

{⍵/⍨∨⌿0=3 5∘.|⍵}  ⍝ 2016.7

{/∘⍵¨(,⍥⊂∘~⍨)0>⍵}  ⍝ 2016.8
{(⍵/⍨m)(⍵/⍨~m←0>⍵)}  ⍝ 2016.8. A little more readable, but not as cool

{1↓¨x⊂⍨⍺∊⍨x←⍵,⍨⊃⍺}  ⍝ 2016.9. Split on a set of delimiters, keeping empty strings.

+.×  ⍝ 2016.10
```

### 2017

```apl
{¯1+2×⍳⍵}  ⍝ 2017.1
{+\2-⍵↑1}  ⍝ 2017.1. Doesn't rely on ⎕IO being 1!

⊢+1=2∘|  ⍝ 2017.2

{360÷⍨⍺×○×⍨⍵÷2}  ⍝ 2017.4

∧/∊∘'ATCG'  ⍝ 2017.5

+/'ACGT'∘.=⊢ ⍝ 2017.7

{2>≢⍵: (≢⍵)⍴0 ⋄ 0,⍨2=/⍵}  ⍝ 2017.9

{(s,s)⍴⍵↑⍨*⍨s←⌈0.5*⍨≢⍵}  ⍝ 2017.10
```

### 2018
```apl
+/⍤≠⌈\  ⍝ 2018.1. Ask how many unique elements there are in the max-scan.

⍝ 2018.1. Number of differing consecutive pairs in the max-scan. Need to add `1` to get the right answer, and special-case empty arrays.
⍝ This is more efficient because there's no hashing involved. We're making use of the structure of the max-scan (strictly increasing). We can use `cmpx` to profile these implementations.
{⍬≡⍵: 0 ⋄ 1+ +/ 2</ ⌈\ ⍵}

⍝ 2018.2
{(⍵-rem),rem←1|⍵}
{⌊ , ⍵-⌊⍵}
⌊ , ⊢ - ⌊  ⍝ Tacit rephrasing of the above
⌊ , 1∘|
0 1∘⊤  ⍝ Coolest solution IMO. Rephrasing the problem as finding the encoding in base 1.

⍝ 2018.3
{{⍺ ('*'⍨¨⍵)} ⌸ , +/¨ ⍳⍵}  ⍝ ⍺⍨¨⍵ which creates an array in the shape of ⍵ and then replaces each element with ⍺. The final result's shape will be different if ⍺ is not a scalar.
{{⍺ ('*'⍨¨⍵)}⌸+⌿1+(,⍵)⊤¯1+⍳×/⍵}  ⍝ Does not generate high-rank arrays as above (which caps us at inputs of length ≤ 15). We use ,⍵ so that encode's left arg is a vector.

{0=⍺: ⍵ ⋄ ((×⍺)×-≢⍵)↑⍵↓⍨-⍺}  ⍝ 2018.7

×2⊥×⍤-  ⍝ 2018.9
≡⍥{' '~⍨s[⍋s←,⍵]}  ⍝ 2018.10
```

### 2019
```apl
{⍵⊂⍨(⍺↑1)⍴⍨≢⍵}  ⍝ 2019.1

{'FDCBA'[0 65 70 80 90⍸⍵]}  ⍝ 2019.2

{m/⍨(∧/0∘<∧9∘>)¨m←(⊂⍵)+,(2 1)(1 2)∘.×∘.,⍨¯1 1}  ⍝ 2019.4. Knight's moves. Good use of outer product

f←(≢∘∪≠≢)¨⊆  ⍝ 2019.5 Nest is cool. If the input is a simple scalar or already nested, does nothing. Otherwise, boxes the result into a scalar 1-item array.

⍝ 2019.6
{⊢/¨⍸⍵∘.∊'1' '2ABC' '3DEF' '4GHI' '5JKL' '6MNO' '7PQRS' '8TUV' '9WXYZ'}
{lut←(¯1+⍳10),(1+⍳8)/⍨4 3 4,⍨5⍴3 ⋄ lut[(⎕D,⎕A)⍳⍵]}  ⍝ As fast as can be, because we're just using a lookup table.

{+/|2-/(⊢,⊃)(+/¯9 ¯11∘○)¨⍵}  ⍝ 2019.8. Using complex numbers to model distances. Nice use of trains too - ⊢,⊃ appends to an array its first element. 
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

### 2020

```apl
⊢⊂⍨128∘>∨191∘<  ⍝ 2020.2 5-train!

{⌽⍣(>/⍵)⊢(⌊/⍵)+0,⍳|-/⍵}  ⍝ 2020.5. Conditional flip! Index up to magnitude of the subtraction, add smaller as an offset, flip the result only if left is larger.

{⍵[⍒⍺=⍵]}`  ⍝ 2020.6

⍝ 2020.7
{b←2⊥⍣¯1⊢⍺ ⍵ ⋄ b[;1]≡∧/b}  ⍝ my initial approach
(∧/≤/)2⊥⍣¯1,  ⍝ Much nicer - why use AND and compare against the original when you can just use ≤

{∧/¯1=2×/×2-/(10∘⊥⍣¯1) ⍵}  ⍝ 2020.8

⍝ 2020.9
⍝ The consecutive comparisons should yield an array that has zero or more 1s followed by zero or more 0s. Such an array has the property
⍝ that it equals its AND-scan. Conversely if an array doesn't satisfy this property then it's not of that form. 
{⍵≡⍬:1 ⋄ (⊢≡∧\)2≤/,⍵}
{(⊢≡∧\)2≤/0,⍵}  ⍝ This works only if the argument is guaranteed to only contain positive numbers. It's a hack to avoid special-casing.
```

### 2021
```apl
{100×(≢⍵)÷⍨+/⍵∊'GC'}  ⍝ 2021.1

⍝ 2021.2
{l←1+≢⍺ ⋄ 0@{l=⍵}⍺⍳⍵}  ⍝ 2021.2
{1+≢⍺}|⍳  ⍝ Courtesy abrudz. Much nicer.

{(/∘⍵)¨↓0=⍺∘.|⍵}∘,  ⍝ 2021.3

4÷⍨(¯2+○1)××⍨  ⍝ 2021.4

⍝ 2021.7. Magic square check
(∧/⊢=⊃)(+⌿),(+/),(+⌿1 1∘⍉∘⌽),(+⌿1 1∘⍉)
∧/⊢=⊃(+⌿),(+/),+/,⍥(1 1∘⍉)∘⌽  ⍝ Rephrasing of the above using ⍥ (over).

|⍤-⍥{0 24 60⊥¯3↑⍵}  ⍝ 2021.8

⍝ 2021.9. Max run length. Useful to know in run-length encoding (RLE).
{{⍺←0 0 2 ⋄ ⍬≡⍵:⊃⍺ ⋄ (⊃⍵)=⍺[3]:(1↓⍵)∇⍨(⍺[1]⌈1+⍺[2]),(1+⍺[2]),⊃⍵ ⋄ (1↓⍵)∇⍨(1⌈⍺[1]),1,⊃⍵}×2-/⍵}  ⍝ My initial attempt. A direct translation of Scheme-like languages. Messy.
{⌈/¯2-/⍸1,⍨1,2≠/×2-/⍵}  ⍝ Nice idea: ¯2-/ flips (⌽) the arguments in the n-wise reduction.
```

### 2022
```apl
{⍵-⌈/¨|⍵-⍳,⍨(2×⍵)-×⍵}  ⍝ 2022.6. A nice trick to set the width of the square: 2⍵-sign(⍵). Yields 0 for 0, 2⍵-1 for everything else.

{∊(≢⍴+/÷≢)¨⍵⊆⍳≢⍵}  ⍝ 2022.7

⍝ 2022.9
{(+⌿÷≢)¨(1+2×⍺),/(⍺⍴⊃⍵),⍵,⍺⍴⊢/⍵}
(+⌿÷≢)¨(1+2×⊣),/(⍴∘⊃,⊢,⍴∘(⊢/))  ⍝ Tacit rephrasing of the above
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

### 2023

```apl
{⍵/⍨⍵=⌈\⍵}  ⍝ 2023.1

(1-∘≢⊣)⌽⍷  ⍝ 2023.2

{a[⍵⍳⍨(-⍺)⌽a←' ',⎕A]}  ⍝ 2023.3. Search for `⍵` in the rotated alphabet and pick the resultant indices from the original alphabet.

{↑⍺[2]{⍺(≠⊆⊢)⍵}¨⍺[1](≠⊆⊢)⍵}  ⍝ 2023.6

{(n÷⍵),⍨n←1∧⍵}  ⍝ 2023.7. Neat idea: The numerator of the rational representation of a fraction ⍵ is 1∧⍵.
```
