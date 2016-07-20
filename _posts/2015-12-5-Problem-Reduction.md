---
layout: post
title: Problem Reduction
published: true
---

This post is a collection of notes I put together when trying to understand the concept of problem reduction. It is more or less taken directly from Ullman and Hopcroft's excellent textbook on Automata Theory.

Note: this post presumes knowledge of the concept of undecidability.

The technique of **problem reduction** is used to say whether a program is undecidable. We know that the question "given a program and some input, does the program print hello world upon receiving the input" is undecidable. Let's call this question P.

When we are given a different question Q and we want to prove that it is undecidable, we can follow the method that was used to prove that P is undecidable. This is a proof from first principles, and it resembles the [diagonal method argument](https://en.wikipedia.org/wiki/Cantor's_diagonal_argument) put forward by Cantor to prove that real numbers are uncountably infinite. But often it is easier to proceed via the method of *reduction*. We look for a way to reduce P to Q. This means proving that P is a special case of Q, or equivalently that being able to solve Q means being able to solve P.

How do we do this? We think of a procedure that constructs a corresponding instance of Q given any instance of P. Then we note that if Q is decidable, so is P - since given any instance of P we have a procedure to turn it into an instance of Q. And since P is known to be undecidable, we can conclude that Q is undecidable as well.

Effectively, we are proving that Q is a harder problem than our original problem P.

### Example

Suppose Q is the question "given a program PROG1 and some input, does the program ever call a function foo() when it is run?" Our goal is to prove that Q is undecidable using the method we discussed above - we start with an arbitrary instance of Q and think of a procedure to turn it into an instance of our original hello-world problem P.

Here is what we will do.

1. In program PROG1, if there is a function named foo(), rename it to something else, and rename all invocations of foo() to this new name as well. It doesn't matter what we rename it to. The point is to avoid a namespace clash.
2. Add a function named foo() to PROG1 with an empty body.
3. At every point in the program where output is done, add a function check() which checks if "hello world" has been printed thus far - and if yes, calls the function foo(). Exactly how check() is implemented isn't important. One way might be to have a global growable array of characters which is pushed into each time a character is printed. check() would then simply check if the array contains the characters "hello world" in that order.

We have modified the initial program PROG1. Let's call this new program PROG2. By our construction, PROG1 and PROG2 behave identically in all respects when given the same input, except that PROG2 always calls foo() in situations where PROG1 prints "hello world". Conversely, if PROG2 calls foo() we can be sure that PROG1 must have printed "hello world". There is no other way in which PROG2 can call foo(), since we designed it that way.

In other words, the statement 'PROG1 prints "hello world"' always has the same truth value as the statement 'PROG2 calls a function named foo()'.

Now if Q were decidable - that is, we could always determine if a program calls a function named foo() given some input - then we could tell, for any input, if PROG2 would call foo(). This means that we could tell, for any input, if PROG1 would print "hello world". This means P would be decidable. But it isn't, and so Q in turn isn't decidable.

To sum up:

1. You have P, which is known to be undecidable.
2. You have Q, and you would like to prove that it is undecidable.
3. You search for a way to convert an arbitrary instance of P into an instance of Q. If you succeed in finding such a way, you have proved that Q is undecidable.

The key word is *arbitrary*. *Any* instance of P should be convertible to an instance of Q. The mapping need not be one-one, but at the very least it should account for every member of P.
