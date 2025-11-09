+++
title = "Deriving the Y combinator"
date = 2015-12-07
+++

This post is an attempt to put my understanding of the intuition behind the Y combinator into words. It is a follow up to my earlier, introductory post on the [untyped lambda calculus](/blog/lambda-calculus).

Recall that the Y combinator is defined like so:

    Y := λ y → (λ x → y (x x)) (λ x → y (x x))

It takes as input a function argument `y` and returns the result of applying `(λ x → y (x x))` to itself. This means that output of `Y R`, for some argument `R`, is `R (Y R)`. Note that both `Y R` and `R (Y R)` are partially applied functions.

In the [previous post](/blog/lambda-calculus) we saw how the Y combinator works. We looked at two examples of problems with recursive formulations and saw how to construct functions `R` such that `Y R`, when applied to the required arguments, yields the desired result. One insight we had was that since `R` is called with itself as the first argument, it must be an `(n+1)`-ary function, where `n` is the number of formal parameters in the recursive formulation.

We also saw *why* the Y combinator is needed. It gives us a way to express recursive functions in languages that do not natively support recursion - like the untyped lambda calculus.

"That great", you say, "but *where did the Y combinator come from*?". Fair question! In the last post we simply presented the Y combinator instead of deriving it from scratch or supplying the intuition behind it.

The latter is what this post will attempt to do.

Our method is as follows.

1. We think of a situation that requires the use of a special kind of function.
2. We write down a crude definition of that function that breaks the rules of the lambda calculus.
3. We then refactor our definition so that it is a valid lambda expression.
4. That special function is our Y combinator.

Let's start with our old friend the factorial function.

    F = λ x → Z n 1 (M n (F (P n)))

(*please read the [previous post](/blog/lambda-calculus) if you aren't sure what this means*)

The problem with this is that `F` is not defined until the expression on the right is fully evaluated, which requires an application of `F`, which requires that `F` is fully evaluated to begin with. Let's break this cycle. Observe that the right hand side can be seen as a function of `F`. Sure, it's a lambda taking a single input `x`, but it could just as well be seen as a function which accepts a single input `F'`, like so:

    G = λ F' → (λ x → Z n 1 (M n (F' (P n))))
    G = λ F' x → Z n 1 (M n (F' (P n)))

and so our previous expression can be written as:

    F = G F

A few things to note about this.

1. There was nothing special about the factorial function. We could just as well have written the above derivation for any other recursive function.
2. The arity of `F` wasn't relevant in our derivation either. `F` could have taken, say, 17 arguments and it wouldn't have made a difference to the logic above.

Now, back to our end result:

    F = G F

This is interesting. `F` is a function that takes a number argument (bear in mind that numbers in the lambda calculus are just functions). `G` is a function that takes as input two arguments: a function `F` and a number argument `x`. `G F` is therefore a partially applied function that takes one more argument as input. This function `G F` is equal [1] to `F`, which is the input to `G`.

In other words, `F` is a fixed point of `G`. So if we know `G`, we can find `F` by finding `G`'s fixed point. How do we do that? Ideally we would have a function `Y` which would take as input another function and returns its fixed point. This is of course the Y combinator that is the subject of this blog post. Given such a function, we could then pass `G` to it and get back `G`'s fixed point `F`. In other words:

    F := Y G

and so

    Y G = G (Y G)
    Y = λ G → G (Y G)

Note that the above line doesn't constitute a formal definition of the Y combinator. Why? Because `Y` appears on the right hand side, and so we have the same issue as we did with `F`.

We need to get rid of `Y` appearing on the right hand side. How?

Well, one example of a function that calls its input on itself is the **U combinator**, which we encountered in the [previous post](http://wenderen.github.io/Notes-Lambda-Calculus).

    U := λ x → x x

Here we have a function `Y` calling itself. It's tempting to use the U combinator in this setting. But then what should we apply the U combinator to? Let's call the unknown argument to the `U` combinator `H`, and try to derive an expression for it. We have:

    Y G = U H
    Y G = H H

where `H` is chosen such that the property `Y G = G (Y G)` is satisfied. Now let's find `H`.

    Y G = G (Y G)
    H H = G (H H)
    H x = G (x x)
    H = λ x → (G (x x))

The key step here was going from the second line above to the third line. We noticed that the second `H` in `H H` was reappearing in `G (H H)` and could therefore be treated as a parameter of the first `H` in `H H`. We then replaced it by a formal parameter `x` and rearranged things some more.

Let's check that `Y G = H H` and `H = λ x → G (x x)` is enough to satisfy our original constraint `Y G = G (Y G)`.

    H = λ x → G (x x)
    H H = (λ x → G (x x)) (λ x → G (x x))
    H H = (λ u → G (u u)) (λ x → G (x x))
    H H = G (λ x → G (x x G)) (λ x → G (x x G))
    H H = G (H H)
    Y G = G (Y G)

Nice, it works out. So:

    Y G = H H
    Y = λ G → H H
    Y = λ G → (λ x → G (x x)) (λ x → G (x x))
    Y = λ y → (λ x → y (x x)) (λ x → y (x x))

which is what we were trying to obtain!

And now we have the insight behind the Y combinator. It's just the end result of refactoring the definition of a fixed point for functions that take other functions as input.

## Footnotes

[1] Note that for two functions to be equal, they must return the same output for every input
