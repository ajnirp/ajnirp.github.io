+++
title = "Problem Reduction"
date = 2015-12-05
+++

This post is a collection of notes I put together when trying to understand the concept of problem reduction. It is more or less taken directly from Ullman and Hopcroft's excellent textbook on Automata Theory.

Note: this post presumes knowledge of the concept of undecidability.

The technique of **problem reduction** can be used to say whether a program is undecidable. We know that the question "given a program and some input, does the program print `hello world` when run?" is undecidable. Let's call this question P.

When we are given a different question Q and we want to prove that it is undecidable, we can follow the method that was used to prove that P is undecidable. This is a proof from first principles, and it resembles the [diagonal method argument](https://en.wikipedia.org/wiki/Cantor's_diagonal_argument) put forward by Cantor to prove that real numbers are uncountably infinite. But often it is easier to proceed via the method of *reduction*. We look for a way to reduce P to Q. This means proving that P is a special case of Q, or equivalently, that being able to solve Q means being able to solve P.

How do we do this? We think of a procedure that takes an instance of Q and turns it into an instance of P. Then we note that if Q is decidable, so is P - since given any instance of Q we have a procedure to turn it into an instance of P.

Effectively, we are proving that Q is a harder problem than our original problem P. If the harder problem is decidable, so is the easier problem. And since the easier problem P _isn't_ decidable, neither is our harder problem Q.

### Example

Suppose Q is the question "given a program PROG1 and some input, does the program ever call a function `foo()` when it is run?" Let's use problem reduction to prove that Q is undecidable. We'll start with an arbitrary instance of Q and think of a procedure to turn it into an instance of our original hello-world problem P.

Here is what we will do.

1. In program PROG1, if there is a function named `foo()`, rename it to something else, and rename all invocations of `foo()` to this new name as well. It doesn't matter what we rename it to. The point is to avoid a namespace clash.
2. Add a function named `foo()` to PROG1 with an empty body.
3. At every point in the program where output is done, add a function `check()` which checks if "hello world" has been printed thus far - and if yes, calls the function `foo()`. Exactly how `check()` is implemented isn't important. One way might be to have a global growable array of characters which is pushed into each time a character is printed. `check()` would then simply check if the array contains the characters "hello world" in that order.

We have modified the initial program PROG1. Let's call this new program PROG2. By our construction, PROG1 and PROG2 behave identically in all respects when given the same input, except that PROG2 always calls `foo()` in situations where PROG1 prints `hello world`. Conversely, if PROG2 calls `foo()` we can be sure that PROG1 must have printed `hello world`. There is no other way in which PROG2 can call `foo()`, since we designed it that way.

In other words, the statement 'PROG1 prints `hello world`' always has the same truth value as the statement 'PROG2 calls a function named `foo()`'.

Now if Q were decidable - that is, we could always determine if a program calls a function named `foo()` given some arbitrary input - then we could determine if PROG1 would print "hello world" given some arbitrary input. This means P would be decidable. But it isn't, and so Q in turn isn't decidable.

The key word is *arbitrary*. *Every* instance of Q should be convertible to an instance of P. The mapping need not be one-one, but at the very least it should account for every member of Q.
