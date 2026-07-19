+++
title = "Knight's moves in APL"
date = 2026-07-18
+++

I've been learning APL recently, and when I found [APL quest](https://apl.quest/) recently, I was addicted. It's a collection of bite-sized problems each meant to be solved with an APL one-liner. In this blog post I'll discuss [2019-4](https://apl.quest/2019/4/), which asks us to return all valid knight moves given a 2-element vector representing the knight's current position on the chessboard.

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
Fun!