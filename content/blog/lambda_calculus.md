+++
title = "Notes on the Lambda Calculus"
date = 2015-07-21
+++

This blog post is an introduction to the lambda calculus. It doesn't assume too much prior knowledge. In fact, I've tried to make this post self-contained. If something is unclear, please let me know!

## Table of Contents

* [Introduction](#introduction)
    * [What is the lambda calculus?](#what-is-the-lambda-calculus)
    * [Examples](#examples)
    * [About this article](#about-this-article)
* [Numbers](#numbers)
* [Arithmetic](#arithmetic)
    * [Successor](#successor)
    * [Addition](#addition)
    * [Multiplication](#multiplication)
* [Logic](#logic)
* [Control flow](#control-flow)
* [Tuples](#tuples)
* [More arithmetic](#more-arithmetic)
    * [Predecessor](#predecessor)
    * [Subtraction](#subtraction)
    * [Division](#division)
* [Comparisons](#comparisons)
* [Recursion](#recursion)
* [Negative numbers](#negative-numbers)
    * [Successor](#successor-1)
    * [Predecessor](#predecessor-1)
    * [Addition](#addition-1)
    * [Subtraction](#subtraction-1)
* [Conclusion](#conclusion)
* [Reference](#reference)

## Introduction

### What is the lambda calculus?

The lambda calculus is a minimal model of computation. It specifies how to define functions and how to apply them to other arguments - which are themselves functions by virtue of the fact that the lambda calculus doesn't specify how to define anything *other than* a function. Despite this minimalism, the lambda calculus is equivalent to a Turing machine.

A **name** or **variable** is an identifier, usually a lowercase letter. A **lambda** is a function with no name. It consists of a list of space-delimited names following the `λ` symbol. Following the argument list is an arrow `→` and the **body** of the lambda, which is an expression.

What is an expression? An **expression** is either a name (defined above), a lambda (defined above) or an application. An expression can have a pair of parentheses `()` surrounding it. This doesn't change anything in the expression, but it can make reading easier sometimes.

A **application** is two expressions written with a space in between. The first expression is said to be applied to the second. In lambda calculus, the convention is to associate from the left. So the application `x y z` is read as "`x` applied to `y`, and the result is then applied to `z`". Alternatively, this can be read as "`x` is applied to both `y` and `z`". Both interpretations are valid and in fact the same. Note that both these interpretations are *different* from this one: `x (y z)`. Can you say why?

### Examples

`λ x y → x y y` is an example of a lambda. The `x y` after the `λ` is the argument list. The `x y y` after the arrow is the body of the lambda. If you were to describe this function in words, you might say "a lambda that takes two arguments, duplicates the second and applies the first to them".

`(λ x y → x y y) f g` is an example of a function application. The lambda we just defined is applied to two variables `f` and `g`, which are themselves functions - after all, everything in the lambda calculus is an anonymous function. To apply a function to some expressions, substitute each expression in the corresponding lambda. In this case, the application yields:

    (λ x y → x y y) f g
    (λ y → f y y) g
    f g g

We proceeded in two steps. After the lambda was applied to `f`, `x` got substituted with `f`. We then had a lambda that accepted only a single argument `y`, duplicated it, and applied `f` to both. Next, this new lambda was applied to `g`. `y` got substituted with `g`, and we were left with `f g g`.

Here's another example. Consider the lambda `λ x → x x`. It's known as the **U combinator**, and all it does is apply its only argument to itself. If we pass the `U` combinator to itself, what do we get?

    (λ x → x x) (λ x → x x)

Since the argument being passed to the first `U` also contains an `x`, we need to first rename it to something else to prevent a name clash. This is known as **alpha renaming**.

    (λ x → x x) (λ x → x x)
    (λ x → x x) (λ y → y y)

So `x` must be substituted by `(λ y → y y)` in the body of the first `U` - which is `x x`. This would yield

    (λ y → y y) (λ y → y y)

which is the `U` combinator applied to itself again. So `U U` is an infinite program - and in fact, since it regenerates itself at every step, it is a [quine](https://en.wikipedia.org/wiki/Quine_(computing))!

### About this article

In this article, we'll look at how the lambda calculus implements some concepts that we expect from programming languages - numbers, arithmetic, logic, control flow, and recursion. It's really interesting to see how in spite of being an extremely minimal language, the lambda calculus is able to support all these concepts by building abstraction upon abstraction. Along the way, we'll try to understand the intuition behind some of these implementations.

## Numbers

`0` is **`λ s z → z`**. It ignores its first argument and returns the second one unmodified. `1` is **`λ s → s (s z)`**. `2` is **`λ s z → s (s (s z))`**. And so on. Essentially, a number in the untyped lambda calculus says "give me a function and another argument, and I'll apply the function this many times to that argument." This is why `0` ignores its first argument - applying a function zero times to a value means simply not changing the value at all.

That's just the whole numbers, though. What about negative integers? We'll get there.

## Arithmetic

### Successor

The successor function is **`S = λ w y x → y (w y x)`**. When applied to a number, it returns the next number, like so:

    S M
    (λ w y x → y (w y x)) M
    λ y x → y (M y x)
    λ y x → y ((λ s z → <s applied to z M times>) y x)
    λ y x → y (<y applied to x M times>)
    λ y x → <y applied to x M+1 times>
    λ s z → <s applied to z M+1 times>
    M+1

Ok, great, it works out. But why does the successor function need three arguments? As it turns out, the difference between M and M+1 is a single extra application of the `s` parameter. So we need one parameter for that - this is the `w` parameter - and two more parameters `y` and `x` to consume the two formal parameters of M! This is the importance of the `(w y x)` bit - we are resolving `M` in the context of `y` and `x` and then applying `y` once more outside the brackets.

### Addition

The successor function also works as an addition operator, because `M S N`, as per our intuition about numbers, is `S applied M times to N`. Adding `M` to `N` is therefore the same as incrementing the number `N` a total of `M` times. Nice - we got two functions for the price of one.

### Multiplication

Multiplication is **`M = λ x y z → x (y z)`**. This is also called the [B combinator](https://en.wikipedia.org/wiki/B,C,K,W_system), but never mind that. How does it work?

    M A B
    (λ x y z → x (y z)) A B
    λ z → A (B z)
    λ z → (λ t → A (B z t))

That last line was a technique known as **eta-conversion**. Eta-conversion says that if `E` is an expression in which `x` is NOT a free variable, then `E` is the same as `λ x → E x`. We're essentially taking an expression `E` and applying it to a variable `x` that is bound by a lambda we just introduced, and which does not itself appear in `E`. In the last line above, we eta-converted by adding in a `t` parameter. Now we can [uncurry](https://en.wikipedia.org/wiki/Currying) the above line to get

    λ z t → A (B z t)
    λ u → (λ z t → A (B z t)) u
    λ u z t → A (B z t) u

in other words, apply `z` to `t` B times. Then, apply that function to `u` A times.	In other words, do the operation "apply `z` to `t` B times" a total of `A` times - but to what? To the variable `u` - which is just a dummy variable we introduced so that `A` would have two arguments! Remember, our whole numbers are represented by functions which take two arguments each.

So, we have: "apply `z` to `t` a total of `A * B` times". And that's just another way of stating the function

    λ z t → <apply z to t A * B times>

which in turn is the number `A * B` in our representation.

## Logic

Yay logic! Specifically, Boolean logic.

To have Boolean logic we need truth and falsity. The value `T` is `λ x y → x` and the value `F` is `λ x y → y` (Kind of like Scheme's `car` and `cdr`, but I digress).

What else do we need? Right, Boolean operations.

1. `AND` is **`λ x y → x y F`**. Why? Let's look at each case. If `x` is `F`, `F y F` is `F` regardless of `y`. If `y` is `F`, it doesn't matter if `x` is `T` (picks the first one) or `F` (picks the second one) - either way, `F` is going to be picked. And if both `x` and `y` are `T`, `T T F` is `T`.

2. `OR` is **`λ x y → x T y`**. The reasoning is similar.

(The other two combinations, by the way, aren't as interesting. `x F y` turns out to be `y AND NOT x`. `x y T` turns out to be `NOT (x AND NOT y)`. Oh well.)

3. 'NOT' is **`λ x → x F T`**. Easy. (`λ x → x T F` on the other hand is just the identity function.)

(Side note: `F` and `0` have the same definition, but you probably already noticed this :) )

## Control flow

We get `if` conditions via the `Z` predicate. The `Z` stands for `zero?` - it takes a single whole number input and returns `T` if the input is `0`, and `F` otherwise. The predicate itself is **`λ x → x F NOT F`** and the way it works is pretty cool. Recall that in our lambda calculus, numbers specify the number of times to apply a function. If we pass `0` to `Z`, we get `0 F NOT F`, which is `NOT F` - since `0` ignores its first argument - which in turn is `T`. If we pass any *other* whole number `M`, we get `M F NOT F`, which resolves as

    M F NOT F
    <apply F to NOT M times> F
    F (F (... (F NOT) ...)) F
    F <something> F
    F

We didn't have to worry about the contents of that `something` - after all, `F` is defined as `λ x y → y`, meaning that it simply discards its first argument and returns the second one.

## Tuples

To represent a pair `(a, b)`, we use the function `λ z → z a b`. We extract its first element with `T` like so:

    (λ z → z a b) T
    T a b
    a

and its second element with `F` like so:

    (λ z → z a b) F
    F a b
    b

(remember what we said earlier about `T` and `F` being like `car` and `cdr` respectively?)

Also, since a pair is just a 2-tuple and there is nothing special about the number 2 in this context, we could very well define an n-tuple `(a1, a2, a3, ..., an)` with the function `z → z a1 a2 a3 ... an`. This is a digression, though. Pairs are enough for what we're about to do next, which is...

## More arithmetic

...subtraction!

### Predecessor

How do you subtract `1` from a number `N` using *only* the tools we've defined thus far? We have addition, multiplication, tuples and boolean logic in our toolkit. Multiplication doesn't seem to be a good choice. Boolean logic would help with branching, as we've seen above, but it's unlikely we'd need to branch just to define subtraction. Addition seems like a good bet - it is after all the inverse of subtraction. And tuples? Let's see.

Keeping with our addition-based line of thought, one idea is to represent `N-1` by adding `N-1` to `0`, or in other words, incrementing `0` a total of `N-1` times using our successor function. How do we represent `N-1`, though? After all, this is what we're aiming for in the first place. The trick is to apply the same function `N` times to a pair, after which one of the pair's elements will equal `N-1`.

Take a look at this function:

    λ p z → z (S (p T)) (p T)

Let's call it `PHI`. Given a pair `λ z → z a b`, it returns the pair `λ z → (S a) a`. So if you pass it a pair `(n, n-1)`, it will return the pair `(n+1, n)`. And if you pass it the pair `(0, 0)`, it will return the pair `(1, 0)`. This is the key to finding the predecessor of a number `N`. We apply `PHI` to `(0, 0)` a total of `N` times. The first time we get `(1, 0)`. After `N-1` more rounds we get `(N, N-1)`. Then we use `cdr` aka `F` to extract the second element of the pair. So our predecessor function `P` is

    λ n → n PHI (λ z → z 0 0) F

Also note that if `n` is `0`, we get `P 0 = F`, and `F` is really `0` in disguise. So the predecessor of `0` is `0` itself. This seems like a silly property, but it will be useful soon.

### Subtraction

Just like addition and the successor operator, subtraction is achieved by repeatedly applying the predecessor operator. `M P N` is "apply `P` to `N` a total of `M` times" - in other words, `N - M`. Note that since the predecessor of `0` is `0`, if `M` is greater than `N` we will at some point be repeatedly applying `P` to `0`, and our final result will be just `0`.

### Division

There isn't a short combinator for division analogous to multiplication that I know of, but division can be formulated as a recursive problem - and we'll see how in a later section.

## Comparisons

To test if a number `x` is greater than or equal to `y`, subtract `x` from `y` by decreasing `y` a total of `x` times. If you're left with `0`, `x` must have been at least as large as `y`. So our function `GE` is:

    λ x y → Z (x P y)

A similar test can be used for `LE`, lesser than or equal to:

    λ x y → Z (y P x)

For `GT`, greater than, we simply use `NOT LE`. Likewise, `LT` is `NOT GE`.

Note by the way that if the predecessor of `0` were not `0`, none of the above definitions would work. See, I told you this property would come in handy.

What about equality? If and only if `x >= y` and `y >= x` both hold, then `x = y`. So our `EQ` function is:

    λ x y → AND (GE x y) (GE y x)

Our not equal (`NE`) function is `NOT EQ`.

## Recursion

In recursion as we know it in most programming languages, a function calls itself within its definition, and an edge case terminates the recursion. Like so:

    factorial 0 = 1
    factorial n = n * factorial (n - 1)

There are cleverer ways to write this function. We could rewrite it in tail-call-recursive form, for example. But fundamentally, the problem is that we are referring to the function name inside the body of the function. How would we do this in the lambda calculus? The whole point of the lambda calculus is that *everything* is expressed by the application of *nameless* functions.

How do you refer to something when it has no name?

Behold:

    λ y → (λ x → y (x x)) (λ x → y (x x))

This function is known as the **Y combinator**. What makes it special? Pass it a function argument `R`:

    Y R
    (λ y → (λ x → y (x x)) (λ x → y (x x))) R
    (λ x → R (x x)) (λ x → R (x x))
    (λ z → R (z z)) (λ x → R (x x))
    R ((λ x → R (x x)) (λ x → R (x x)))
    R ((λ x → R (x x)) (λ x → R (x x)))
    R (Y R)

Look at the first and last lines. `Y R = R (Y R)`. The right hand side is what we were looking for - a way to call R from within itself. This is what makes the Y combinator useful. It lets us use recursion in the untyped lambda calculus. If we want to recursively call a function from within itself, we simply pass it as an argument to the Y combinator.

What could be an example of a function that we might want to call recursively? When college students are taught about recursion a common pedagogical example is the `factorial` function mentioned above. Let's set that aside in favour of an even simpler example that the [reference paper for this article](http://www.inf.fu-berlin.de/lehre/WS03/alpi/lambda.pdf) uses - **summing the first `n` numbers**.

The sum of the first `n` numbers is `n` plus the sum of the first `n-1` numbers. The base case is when `n` is `0`, in which case the sum is simply `0`.

Now let's think about how our recursive function should look.

First off: how many arguments should it take? We have only one numerical parameter `n`, so "one" is a good first guess. But we're working within the constraints of the Y combinator. Since `Y R` expans to `R (Y R)`, this means that `R` must take at least one more parameter, corresponding to this `(Y R)`. What does this other parameter represent? The recursive call of course!

So let's try to write our recursive function with two arguments.

    λ r n → <body of our function - what do we put here?>

Here, `r`, the first parameter, represents the recursive call we will make somewhere in the body of our function `R`. `n` is the number upto which we want to sum, starting from `0`.

In the description of our recursion, we have an `if` condition - is `n` zero or not. We only know of one way to express branching - the `zero?` predicate we defined earlier. It seems to be a good fit.

    λ r n → Z n 0 <body of our function - what do we put here?>

If `n` is `0`, we get

    λ r n → T 0 <body>
    0

and if not,

    λ r n → F 0 <body>
    <body>

which is what we want. Nice. Now let's think of what we should put in the body. Our recursive call will return the sum from `0` to `n-1`. We need to add `n` to that. So this should work:

    λ r n → Z n 0 (n S r)

...er, wait a minute. `r` isn't the *result* of our recursive call, it's just the function that makes the recursive call. So it needs an argument of its own. In this case, the argument should be `n-1`, which in our lambda calculus is `P n`. And so we get:

    λ r n → Z n 0 (n S (r (P n)))

Now let's try to run our function. On paper, of course.

    Y R 3
    R (Y R) 3
    (λ r n → Z n 0 (n S (r (P n)))) (Y R) 3
    Z 3 0 (3 S ((Y R) (P 3)))
    Z 3 0 (3 S ((Y R) 2))
    F 0 (3 S ((Y R) 2))
    (3 S ((Y R) 2))
    3 S ((Y R) 2)
    3 S (Y R 2)

We started out with `Y R 3` and ended up with `3 S (Y R 2)`. There's the recursion we wanted!

Now let's run the above step two more times. As you can guess, we'll get `3 S (2 S (1 S (Y R 0)))`. `Y R 0` would expand to give us

    Y R 0
    R (Y R) 0
    (λ r n → Z n 0 (n S (r (P n)))) (Y R) 0
    Z 0 0 (0 S ((Y R) (P 0)))
    T 0 (0 S ((Y R) (P 0)))
    0

(See how `Z 0` broke the recursion? Good old `Z`. Where would we be without it?)

Our final result is therefore `3 S (2 S (1 S 0))`, that is, `6`. Hooray! Now let's do the **factorial** function. What changes do we need to make?

1. The base case is still `n = 0`. But now the return value of the base case should be `1`.

2. We have to multiply `n` with the result of the recursive call.

And so our function becomes:

    λ r n → Z n 1 (M n (r (P n)))

For kicks, let's also do **division**. Again, the pattern is the same, with only minor variations.

    λ r x y → (GT x y) 0 S (r x (x P y))

`x` and `y` are the two arguments, and we want to find the quotient when `y` is divided by `x`. The base case is that `x` is greater than `y` - in this case, the quotient is zero. Otherwise, the quotient equals `1` plus the quotient when `y - x` is divided by `x`.

## Negative numbers

Negative integers are represented with pairs: we use a pair `λ z → z a b` to represent the number `a - b`. This of course requires that we change the definitions of our successor / addition and predecessor / subtraction functions so that they can work with pairs. Let's do that.
Raúl Rojas wrote a beautiful [paper](http://www.inf.fu-berlin.de/lehre/WS03/alpi/lambda.pdf) back in 1997 introducing the [lambda calculus](https://en.wikipedia.org/wiki/Lambda_calculus). The paper explains the **untyped lambda calculus** from first principles in very plain language and was the only text that really made me understand the basics. This blog post is a **collection of notes to accompany the paper** - essentially, a summary of the paper mixed in with the insights I had while trying to understand it, as well as solutions to a couple of the easier problems posed at the end.
### Successor

    S' = λ p → (λ z → z (S (p T)) (p F))
       = λ p z → z (S (p T)) (p F)

The meaning is clear enough. The successor of `a - b` is `(a + 1) - b`. Also, note that the first line expresses our intent more clearly - take a pair and return another pair whose values are derived from the original pair. The second line is just an uncurried version of the first.

What else? Right, predecessor. Here we go:

### Predecessor

    P' = λ p → (λ z → z (p T) (S (p F)))
       = λ p z → z (p T) (S (p F))

Pretty much the same as above. By the way - would `λ p z → z (P (p T)) (p F)` be a valid definition for `P'`? *(Hint: consider what happens when the first element of the pair is `0`)*

### Addition

`(a - b) + (c - d) = (a + c) - (b + d)`, so:

    ADD = λ p q → (λ z → z ((p T) S (q T)) ((p F) S (q F)))
        = λ p q λ z → z ((p T) S (q T)) ((p F) S (q F))

### Subtraction

Similar logic. `(a - b) - (c - d) = (a - c) - (b - d)`, so:

    SUB = λ p q → (λ z → z ((p T) P (q T)) ((p F) P (q F)))
        = λ p q λ z → z ((p T) P (q T)) ((p F) P (q F))

## Conclusion

The above was a brief introduction to the lambda calculus. We haven't touched upon some other interesting topics, like how lists are represented in the lambda calculus, and how one can write functions to operate on such lists. More on this to come!

## Reference

Raúl Rojas wrote a beautiful [paper](http://www.inf.fu-berlin.de/lehre/WS03/alpi/lambda.pdf) back in 1997 introducing the [lambda calculus](https://en.wikipedia.org/wiki/Lambda_calculus). The paper explained the **untyped lambda calculus** from first principles in very plain language and was the only text that really made me understand the basics. This blog post began as a collection of notes to accompany the paper, and grew as I added in insights I had while trying to understand the paper, as well as solutions to a couple of the easier problems posed at the end.
