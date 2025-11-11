+++
title = "A backtracking solver for LinkedIn Tango" 
date = 2025-11-09
+++

The other day I wrote a solver for the [LinkedIn Tango](https://www.linkedin.com/games/tango/) puzzle. This blog post explains a few key ideas behind the implementation. If you'd like to dig through the code, it's [here](https://github.com/ajnirp/tango-rs).

## Primer

For those unfamiliar with LinkedIn's games, the company publishes daily online games inspired by (I assume) Wordle and similar games, which first became very popular during the COVID-19 pandemic lockdowns.

[Tango](https://www.linkedin.com/games/tango/) as a puzzle is very reminiscent of the classic one-player puzzle game [Sudoku](https://en.wikipedia.org/wiki/Sudoku). For example:

* You have a grid with some cells "set" to a value, and several "empty" cells. 
* Each empty cell must be set (by you) to solve the puzzle.
* Every value in a "set" cell comes from a fixed set of values. In Sudoku, it's the digits `1` through `9`. In Tango, it's either a "sun" or a "moon".
* Certain game rules enforce constraints on the grid. By extension, these rules influence what values a cell can take.
  * For example, in Sudoku, no row may contain duplicate digits. In Tango, every row must contain an equal number of "suns" and "moons".
* Gameplay involves a player looking at the board, deducing what value an empty cell must take, filling that cell in, and then repeating the previous steps until the grid is filled in.

For a full list of rules, please refer to the [Tango page](https://www.linkedin.com/games/tango/).

## Writing a solver

I figured it would be fun to write a solver. My first thought was to write a solver that would apply one of several heuristics one after the other until all cells were filled up. Some examples of heuristics:

* When you see a cell with two "suns" on either side of it, fill it with a "moon". Likewise if there were two "moons".
* Say the grid width and height are `N`. When you see a row or column with `N / 2` "suns" in total, fill all other cells with "moons".

I wrote up a few of these heuristics and was able to run them on several Tango grids. A couple times I had to add new, cleverer heuristics to account for harder puzzles that were otherwise unsolvable with the heuristics I'd written so far. Naturally, this "expert knowledge" approach was painful to write and required a lot of worrying about loop iterations and edge cases around array bounds.

Which made me take a step back and think: since Tango is quite reminiscent of Sudoku, and Sudoku is solvable by simple backtracking methods, why not do the same for Tango?

I went ahead and did that, and it was indeed a simpler approach to implement. [Here's the code repository](https://github.com/ajnirp/tango-rs). There's lots of boilerplate code in there, but the heart of the code is in `solver.rs`. Specifically, this function:

```rust,linenos
fn helper(board: &mut Board) -> bool {
    let i = board.next_unsolved();
    if i == None {
        return true;
    }
    let i = i.unwrap();
    board.mark_solved();
    for new in 0..2 {
        // set a value only if it's safe to do so
        if can_set(board, i, new) {
            board.set_index(i, new);
            if helper(board) {
                return true;
            }
            board.set_index(i, 2);
        }
    }
    board.mark_unsolved();
    false
}
```

This is a classic backtracking formulation. Pick an empty cell and ask if we can set it to a particular value. If yes, recurse with that value set. At some point, if the search fails, abandon that entire search branch, and instead try the next value (in Tango, there are only two values: `0` representing "sun" and `1` representing "moon").

The "can we set this empty cell to a particular value" logic is a direct translation into code of the game's rules:

```rust,linenos
fn can_set(board: &Board, i: usize, new: u8) -> bool {
    let (drs, dcs) = ([0usize, 1], [1usize, 0]);
    let side = board.side();
    let (r, c) = (board.row(i), board.col(i));

    for v in 0..2 {
        let (dr, dc) = (drs[v], dcs[v]);
        let (mut _r, mut _c) = (r * dc, c * dr);

        // `one` = the value two indices before `curr`
        // `two` = the value one index before `curr`
        let (mut one, mut two) = (255u8, 255u8);

        // number of cells in the row/col that are already equal to `new`
        let mut num_existing = 0;

        for _ in 0..side {
            let curr = if (_r, _c) == (r, c) { new } else { board.at(_r, _c) };

            // three consecutive identical not allowed
            if one == two && two == curr && curr != 2 {
                return false;
            }
            one = two;
            two = curr;

            if curr == new { num_existing += 1; }

            _r += dr;
            _c += dc;
        }
        // no more than `side / 2` in a row or column
        // for example, for a 8x8 board, no more than 4 in a row or column
        if num_existing > side / 2 {
            return false;
        }
    }

    // abide by constraints
    // north, east, south, west
    let _drs = [-1i16, 0, 1, 0];
    let _dcs = [0i16, 1, 0, -1];
    let constraint = board.constraint_at(r, c);
    for j in 0..8 {
        if constraint & (1 << j) == 0 { continue; }

        let _nr = (r as i16) + _drs[j % 4];
        let _nc = (c as i16) + _dcs[j % 4];
        if _nr < 0 || _nc < 0 { continue; }

        let (nr, nc) = (_nr as usize, _nc as usize);
        if !board.inside(nr, nc) { continue; }

        let nbr_val = board.at(nr, nc);
        if nbr_val == 2 { continue; }

        if (j < 4 && nbr_val == new) || (j >= 4 && nbr_val != new) {
            continue;
        }
        return false;
    }

    true
}
```

That's a long function, but all it's doing is verifying that:

* There is no row or cell with 3 consecutive occurrences of the same value.
* There is no row or cell with more than `N / 2` occurrences of the same value.
* Every "x" and "=" wall constraint is satisfied.

I set up a simple test suite, some quick parsers to translate a string representation of a game into a more structured representation, and verified that the approach could handle every single game I threw at it, no matter how tough it was for a human to solve.

## Future work

A natural follow-up is: how are these puzzles generated? Our basic requirement is: we want to generate games that a human can solve **without guesswork**. A prerequisite for not needing to guess is that a game should permit only one solution. Answering _that_ question feels like an extension of the solver above. At a high level, it might look something like this:

* Modify the solver to check whether a given input game has a unique solution, no solution, or more than one solution.
* Take a blank board, give it a few random wall constraints, and fill in a few cells at random. This is a candidate game.
* Run the modified solver on the candidate. If there's a unique solution, we can stop here — we found a game that has a unique solution.
* If there is no solution, start afresh. If there are multiple solutions, fill in one more empty cell and then recheck for solution uniqueness. Keep doing this until we get to a successful candidate.

There's a wrinkle here. Having a unique solution is a necessary condition for a game to be solvable without guesses. But is it a _sufficient_ condition? I'm not sure. Something to think about.