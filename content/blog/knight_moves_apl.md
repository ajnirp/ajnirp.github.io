+++
title = "Knight's moves in APL"
date = 2026-07-18
+++

I've been learning APL recently, and when I found [APL quest](https://apl.quest/) recently, I was addicted. It's a collection of bite-sized problems each meant to be solved with an APL one-liner. In this blog post I'll discuss my solution to [2019-4](https://apl.quest/2019/4/), which asks us to return all valid knight moves given a 2-element vector representing the knight's current position on the chessboard.

```apl
      ]box ON
      (your_function)5 4
┌───┬───┬───┬───┬───┬───┬───┬───┐
│3 3│3 5│4 2│4 6│6 2│6 6│7 3│7 5│
└───┴───┴───┴───┴───┴───┴───┴───┘
```
My solution is
```apl
{m/⍨(∧/0∘<∧9∘>)¨m←(⊂⍵)+,(2 1)(1 2)∘.×∘.,⍨¯1 1}
```

which I like because it uses the outer product `∘.` twice. First with the selfie operator:
```apl
      ∘.,⍨1 ¯1
┌────┬─────┐
│1 1 │1 ¯1 │
├────┼─────┤
│¯1 1│¯1 ¯1│
└────┴─────┘
```
And then to obtain all possible knight's move deltas:
```apl
      (2 1)(1 2)∘.×∘.,⍨¯1 1
┌─────┬────┐
│¯2 ¯1│¯2 1│
├─────┼────┤
│2 ¯1 │2 1 │
└─────┴────┘
┌─────┬────┐
│¯1 ¯2│¯1 2│
├─────┼────┤
│1 ¯2 │1 2 │
└─────┴────┘
```
Then we ravel the above, add to it the scalar-ified original position, and filter the result for all positions that contain only valid cell indices.
```apl
      {m/⍨(∧/0∘<∧9∘>)¨m←(⊂⍵)+,(2 1)(1 2)∘.×∘.,⍨¯1 1}1 1
┌───┬───┐
│3 2│2 3│
└───┴───┘
```
And we're done.

abrudz [presents](https://www.youtube.com/watch?v=K40CsPxYohM) a very cool solution in his video:

```apl
{⍸5=+⌿¨×⍨|(⍳8 8)-⊂⍵}
```
with the idea to select only squares whose squared Euclidean distance from the input is 5. I profiled the two methods:
```apl
      f←{⍸5=+⌿¨×⍨|(⍳8 8)-⊂⍵}
      g←{m/⍨(∧/0∘<∧9∘>)¨m←(⊂⍵)+,(2 1)(1 2)∘.×∘.,⍨¯1 1}
      'cmpx'⎕cy'dfns'
      w←?1e2⍴⊂8 8
      cmpx 'f¨ w' 'g¨ w'
  f¨ w → 1.1E¯3 |   0% ⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕
* g¨ w → 4.8E¯4 | -55% ⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕
```

and found that `g` is faster. This feels intuitive: generating 64 options and then eliminating (at least) 56 of them should be slower than directly generating 8 options. For a slight speed up we can store the deltas in a variable and look it up:
```apl
      deltas←,(2 1)(1 2)∘.×∘.,⍨¯1 1
      h←{m/⍨(∧/0∘<∧9∘>)¨m←(⊂⍵)+deltas}  ⍝ NB. same definition as g, but with a variable lookup
      cmpx 'g¨ w ' 'h¨ w'
  g¨ w  → 5.0E¯4 |   0% ⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕
  h¨ w  → 3.1E¯4 | -39% ⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕⎕
```

which is faster but not impressively so.