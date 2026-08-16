+++
title = "Chess moves in APL"
date = 2026-08-16
+++

I've been learning APL recently, and when I found [APL quest](https://apl.quest/) recently, I was addicted. It's a collection of bite-sized problems each meant to be solved with an APL one-liner. In this blog post I'll discuss my solution to [2019-4](https://apl.quest/2019/4/), which asks us to return all valid knight moves given a 2-element vector representing the knight's current position on an empty chessboard.

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

How about rooks and bishops? I find the elimination idea yields some very readable solutions:

```apl
      {c~⍨⍸0∊¨(⍳8 8)-c←⊂⍵}4 5  ⍝ rook
┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
│1 5│2 5│3 5│4 1│4 2│4 3│4 4│4 6│4 7│4 8│5 5│6 5│7 5│8 5│
└───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
      {c~⍨⍸(=/)¨|(⍳8 8)-c←⊂⍵}4 5  ⍝ bishop
┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
│1 2│1 8│2 3│2 7│3 4│3 6│5 4│5 6│6 3│6 7│7 2│7 8│8 1│
└───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
```

The elimination idea for knights looks similar, but lacks the step at the end where we remove the input. Let's add that step anyway, with no loss of correctness, so that it looks more like the rook / bishop functions:

```apl
      {c~⍨⍸5=+⌿¨×⍨|(⍳8 8)-c←⊂⍵}4 5
┌───┬───┬───┬───┬───┬───┬───┬───┐
│2 4│2 6│3 3│3 7│5 3│5 7│6 4│6 6│
└───┴───┴───┴───┴───┴───┴───┴───┘
```

Now we have three very similar-looking functions.

```apl
{c~⍨⍸0∊¨(⍳8 8)-c←⊂⍵}  ⍝ rook
{c~⍨⍸(=/)¨|(⍳8 8)-c←⊂⍵}  ⍝ bishop
{c~⍨⍸5=+⌿¨×⍨|(⍳8 8)-c←⊂⍵}  ⍝ knight
```

Let's extract out the common part, which is "subtract, apply a filter, apply a where `⍸`, and remove the input", and encode the piece-specific elimination logic separately.

```apl
moves←{c~⍨⍸⍺⍺(⍳8 8)-c←⊂⍵}
knight←{5=+⌿¨×⍨|⍵}
king←{2≥+⌿¨×⍨⍵}  ⍝ similar to knight
rook←0∘∊¨
bishop←=/¨|
queen←rook∨bishop
```

I love how the definition of `queen` is simply a Boolean OR over two other predicates, expressed as a fork. We can test this out:

```apl
      king moves 4 5
┌───┬───┬───┬───┬───┬───┬───┬───┐
│3 4│3 5│3 6│4 4│4 6│5 4│5 5│5 6│
└───┴───┴───┴───┴───┴───┴───┴───┘
      knight moves 4 5
┌───┬───┬───┬───┬───┬───┬───┬───┐
│2 4│2 6│3 3│3 7│5 3│5 7│6 4│6 6│
└───┴───┴───┴───┴───┴───┴───┴───┘
      rook moves 4 5
┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
│1 5│2 5│3 5│4 1│4 2│4 3│4 4│4 6│4 7│4 8│5 5│6 5│7 5│8 5│
└───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
      bishop moves 4 5
┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
│1 2│1 8│2 3│2 7│3 4│3 6│5 4│5 6│6 3│6 7│7 2│7 8│8 1│
└───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
      queen moves 4 5
┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
│1 2│1 5│1 8│2 3│2 5│2 7│3 4│3 5│3 6│4 1│4 2│4 3│4 4│4 6│4 7│4 8│5 4│5 5│5 6│6 3│6 5│6 7│7 2│7 5│7 8│8 1│8 5│
└───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
```

I think this is a good place to stop. A few things not covered in this article:

1. Pieces not being able to move to a square because another piece is blocking the way.
1. Pieces capturing opposing pieces.
1. Pawn movement, including en passant.
1. Castling - checking whether it's possible, and implementing castling assuming it's possible.