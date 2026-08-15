+++
title = "Validating a Sudoku board in APL"
date = 2026-08-12
+++

Given a (possibly partially) filled-in Sudoku board, how can you tell if the board is valid? Assume our input is a vector of `81` numbers, with `0` representing blank cells and filled-in cells carrying one of the numbers `1` through `9`. Here's how I'd do it in APL, as a beginner to the language.

```apl
board←5 3 0 0 7 0 0 0 0 6 0 0 1 9 5 0 0 0 0 9 8 0 0 0 0 6 0 8 0 0 0 6 0 0 0 3 4 0 0 8 0 3 0 0 1 7 0 0 0 2 0 0 0 6 0 6 0 0 0 0 2 8 0 0 0 0 4 1 9 0 0 5 0 0 0 0 8 0 0 7 9
box←3⌿3/3 3⍴⍳9
all←{∧/∧/1≥⍵}
occur←9 9 9⍴(⍳9)∘.=board
∧/ (all+/occur) (all+/[2]occur) (all+/(⍳9)∘.=board[9 9⍴⍋,box])
```

The final line evaluates to `1` if the Sudoku board is valid, else `0`. Let's dissect the solution. First we create our input board. `all` returns `1` if all elements in its input table are `0` or `1`. `box` is a `9×9` table with each cell carrying the index of the box it lies in:

```apl
      box
1 1 1 2 2 2 3 3 3
1 1 1 2 2 2 3 3 3
1 1 1 2 2 2 3 3 3
4 4 4 5 5 5 6 6 6
4 4 4 5 5 5 6 6 6
4 4 4 5 5 5 6 6 6
7 7 7 8 8 8 9 9 9
7 7 7 8 8 8 9 9 9
7 7 7 8 8 8 9 9 9
```

`occur` is a 3D array in which each major cell is a `9×9` mask of the occurrences of each number. For example, the 5th major cell of `occur` tells you where all the `5`s are:

```apl
      (9 9⍴board) (occur[5;;])
┌─────────────────┬─────────────────┐
│5 3 0 0 7 0 0 0 0│1 0 0 0 0 0 0 0 0│
│6 0 0 1 9 5 0 0 0│0 0 0 0 0 1 0 0 0│
│0 9 8 0 0 0 0 6 0│0 0 0 0 0 0 0 0 0│
│8 0 0 0 6 0 0 0 3│0 0 0 0 0 0 0 0 0│
│4 0 0 8 0 3 0 0 1│0 0 0 0 0 0 0 0 0│
│7 0 0 0 2 0 0 0 6│0 0 0 0 0 0 0 0 0│
│0 6 0 0 0 0 2 8 0│0 0 0 0 0 0 0 0 0│
│0 0 0 4 1 9 0 0 5│0 0 0 0 0 0 0 0 1│
│0 0 0 0 8 0 0 7 9│0 0 0 0 0 0 0 0 0│
└─────────────────┴─────────────────┘
```

The shape of `occur` is `9×9×9`. The first `9` represents the number of digits. The next two represent the board dimensions.

And now we can tackle the final line. It's a Boolean AND over three values: are the rows valid, are the columns valid, and are the boxes valid.

* `all+/occur` asks if every row sum for every major cell is at most `1`. `/` reduces along the last dimension, which in `occur` is the individual rows for each mask.
* `all+/[2]occur` asks if every _column_ sum for every major cell is at most `1`. `/[2]` reduces along the second dimension, which in `occur` is the individual columns for each mask.

The fun part is the final Boolean. We flatten and grade `box` to get a sort order representing our boxes, and reshape it to a table:

```apl
      9 9⍴⍋,box
 1  2  3 10 11 12 19 20 21
 4  5  6 13 14 15 22 23 24
 7  8  9 16 17 18 25 26 27
28 29 30 37 38 39 46 47 48
31 32 33 40 41 42 49 50 51
34 35 36 43 44 45 52 53 54
55 56 57 64 65 66 73 74 75
58 59 60 67 68 69 76 77 78
61 62 63 70 71 72 79 80 81
```

We then index `board` with the tabular indices. This yields a table in which each row represents the boxes of the input board!

```apl
      (9 9⍴board) (board[9 9⍴⍋,box])
┌─────────────────┬─────────────────┐
│5 3 0 0 7 0 0 0 0│5 3 0 6 0 0 0 9 8│
│6 0 0 1 9 5 0 0 0│0 7 0 1 9 5 0 0 0│
│0 9 8 0 0 0 0 6 0│0 0 0 0 0 0 0 6 0│
│8 0 0 0 6 0 0 0 3│8 0 0 4 0 0 7 0 0│
│4 0 0 8 0 3 0 0 1│0 6 0 8 0 3 0 2 0│
│7 0 0 0 2 0 0 0 6│0 0 3 0 0 1 0 0 6│
│0 6 0 0 0 0 2 8 0│0 6 0 0 0 0 0 0 0│
│0 0 0 4 1 9 0 0 5│0 0 0 4 1 9 0 8 0│
│0 0 0 0 8 0 0 7 9│2 8 0 0 0 5 0 7 9│
└─────────────────┴─────────────────┘
```

Then, as before, we can obtain an `occur`-like mask with:

```apl
(⍳9)∘.=board[9 9⍴⍋,box]
```

No reshaping required here, since `board[9 9⍴⍋,box]` was already of shape `9×9` and not `81`. Then, as before, we validate the "row sums", which are actually the input board's box sums:

```apl
all+/(⍳9)∘.=board[9 9⍴⍋,box]
```

And this, AND-ed together with the row sum validation column sum validation gives us our overall answer:

```apl
∧/ (all+/occur) (all+/[2]occur) (all+/(⍳9)∘.=board[9 9⍴⍋,box])
```

which we can make slightly shorter by pulling out the `all`:

```apl
∧/all¨ (+/occur) (+/[2]occur) (+/(⍳9)∘.=board[9 9⍴⍋,box])
```

Putting it all together:

```apl
box←3⌿3/3 3⍴⍳9
all←{∧/∧/1≥⍵}
occur←9 9 9⍴(⍳9)∘.=board
∧/all¨ (+/occur) (+/[2]occur) (+/(⍳9)∘.=board[9 9⍴⍋,box])
```

The coolest thing about this solution, I think, was going from `box` to the board box indices via a ravel and grade — just an example of how flexible `⍋` can be. For a problem that calls for a similar usage of `⍋`, check out [APL Quest 2016.4](https://apl.quest/2016/4/).