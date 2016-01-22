---
layout: post
title: Streams, Rulers and Laziness
permalink: Ruler-Series
published: true
---

This blog post discusses the solution to one of the exercises in an online course I've been doing in my spare time.

##Problem Statement

The context of the problem is learning to use lazy evaluation. We start by defining a data structure called a `Stream`. It is a generic list similar to the built-in one in Haskell, except that it is necessarily infinite. It is defined like so:

    data Stream a = S a (Stream a)

Simple enough. A `Stream` is an element of type `a` along with another `Stream` whose elements are also of type `a`. The `S` constructor is basically like `cons`.

We can define utility functions for working with `Stream`s analogous to the ones in the `Prelude`.

    streamRepeat :: a -> Stream a
    streamRepeat n = S n $ streamRepeat n

    streamMap :: (a -> b) -> Stream a -> Stream b
    streamMap f (S x y) = S (f x) (streamMap f y)

For ease of debugging, it also helps to derive the `Show` typeclass for `Stream` to print the first, say, 20 elements.

    instance Show a => Show (Stream a) where
        show s = show $ take 20 $ streamToList s

Now for the problem. The **ruler series** is defined as the series in which the *n<sup>th</sup>* element is the largest power of 2 which evenly divides *n*. It looks like this:

    0, 1, 0, 2, 0, 1, 0, 3, 0, 1, 0, 2, 0, 1, 0, 4, ...

Question: How do you represent the `ruler` series as a `Stream`?

##A First Attempt

The assignment helpfully suggests:

> define a function `interleaveStreams` which alternates the elements from two streams. Can you use this function to implement `ruler` in a clever way that does not have to do any divisibility testing?

We know what `interleaveStreams` should look like. It should take two streams `[x1, x2, ...]`, `[y1, y2, ...]` and return the stream `[x1, y1, x2, y2, ...]`. How does this help us write a definition for `ruler`? Well, every alternate element of `ruler`, starting with the first, is a `0`. So `ruler` might be written as:

    interleaveStreams (streamRepeat 0) <???>

The only question now is, what is `<???>`. This isn't too hard to answer. If we remove the `0`s from `ruler`, we get this sequence:

    1, 2, 1, 3, 1, 2, 1, 4, ...

which, if you look at it carefully, is just the original `ruler` series with `1` added to every element - in other words, `streamMap (+1) ruler`!

We now have a definition for `ruler`:

    ruler :: Stream Integer
    ruler = interleaveStreams (streamRepeat 0) (streamMap (+1) ruler)

All that remains is to define `interleaveStreams`, and we are done. How do we do that? My first instinct was to define it like so:

    interleaveStreams :: Stream a -> Stream a -> Stream a
    interleaveStreams (S x y) (S x' y') = S x (S x' (interleaveStreams y y'))

Then I headed over to the `ghci` prompt, loaded my definitions in, pressed `ruler` and waited expectantly.

And waited. And waited some more.

![Me at the prompt](https://s-media-cache-ak0.pinimg.com/736x/bb/43/57/bb435733159f4040f1c68c79f9dc7e8e.jpg)

##Refining our solution

...as you can guess, `ghci` was sent into an infinite loop trying to evaluate `ruler`. Why? Let's unravel the evaluation of `ruler` step by step. Keep in mind that Haskell is *lazy*, which means that

1. it evaluates arguments to a function only when they need to be evaluated
2. it only evaluates them enough to be able to pattern-match them

Off we go!

    ruler
    interleaveStreams (streamRepeat 0) (streamMap (+1) ruler)

`interleaveStreams` expects two arguments of the form `(S x y)` and `(S x' y')`. Since this is not the case, it attempts to evaluate each of its arguments until they can be represented in this form.

    interleaveStreams (streamRepeat 0) (streamMap (+1) ruler)
    interleaveStreams (S 0 (streamRepeat 0)) (streamMap (+1) ruler)
    interleaveStreams (S 0 (streamRepeat 0)) (streamMap (+1) (interleaveStreams (streamRepeat 0) (streamMap (+1) ruler)))

We've made partial progress. The first argument to the outermost `interleaveStreams` is of the form `(S x y)`. The second argument isn't, though. So we must evaluate it.

    streamMap (+1) (interleaveStreams (streamRepeat 0) (streamMap (+1) ruler))

`streamMap` also expects its second argument to be of the form `(S x y)`. Again, it isn't. So we must evaluate it first before we can evaluate the call to `streamMap`.

    interleaveStreams (streamRepeat 0) (streamMap (+1) ruler)

Wait a minute, didn't we just see this *exact* expression? How do we fix this?

Well, the main reason we ran into an infinite loop above is that we were forced to evaluate the second argument to `interleaveStreams`, which eventually ended up regenerating the original expression. It would be nice if we could recurse **without** evaluating the second argument. We can't realy avoid evaluating the first argument, since intuitively, *something* needs to be destructured in order for things to move forward. In any case, the first argument to `interleaveStreams` is `streamRepeat 0` - and that is trivially easy to destructure.

So we need to write a definition for `interleaveStreams` that forces evaluation of *only* the first argument. In other words, our definition is constrained to look like this:

    interleaveStreams :: Stream a -> Stream a -> Stream a
    interleaveStreams (S x y) w = <???>

How do we complete the definition? Observe that interleaving the stream `(S a1 x)` into the stream `w` is the same as prepending `a1` to the result of interleaving `w` into `x`:

    a1    a2    a3    a4
       b1    b2    b3    b4

    b1    b2    b3    b4
       a2    a3    a4

In the above text diagram, `x` is the stream `[a2, a3, a4, ...]`, and `w` is the stream `[b1, b2, b3, ...]`. It's easy to see that `(S a1 x)` spliced into `w` is the same as `w` spliced into `x`. with the only difference being that the former has an extra `a1` at the beginning. So our new definition for `interleaveStreams` is:

    interleaveStreams :: Stream a -> Stream a -> Stream a
    interleaveStreams (S x y) w = S x (interleaveStreams w y)

This recursive definition is better than our previous one because it doesn't force Haskell to evaluate the second argument right away. Yay!

We can verify that this definition "works" - that is, the interpreter is able to lazily evaluate the `ruler` stream without invoking an infinite loop. Let's re-run the evaluation procedure again and see how things have changed.

    ruler
    interleaveStreams (streamRepeat 0) (streamMap (+1) ruler)
    interleaveStreams (S 0 (streamRepeat 0)) (streamMap (+1) ruler)
    S 0 (interleaveStreams (streamMap (+1) ruler) (streamRepeat 0))
    S 0 (interleaveStreams (streamMap (+1) (interleaveStreams (streamRepeat 0) (streamMap (+1) ruler))) (streamRepeat 0))
    S 0 (interleaveStreams (streamMap (+1) (interleaveStreams (S 0 (streamRepeat 0)) (streamMap (+1) ruler))) (streamRepeat 0))
    S 0 (interleaveStreams (streamMap (+1) (S 0 (interleaveStreams (streamMap (+1) ruler) (streamRepeat 0)))) (streamRepeat 0))
    S 0 (interleaveStreams (S (+1 0) (streamMap (+1) (interleaveStreams (streamMap (+1) ruler) (streamRepeat 0)))) (streamRepeat 0))
    S 0 (interleaveStreams (S 1 (streamMap (+1) (interleaveStreams (streamMap (+1) ruler) (streamRepeat 0)))) (streamRepeat 0))
    S 0 (S 1 (interleaveStreams (streamRepeat 0) (streamMap (+1) (interleaveStreams (streamMap (+1) ruler) (streamRepeat 0)))))
    S 0 (S 1 (interleaveStreams (S 0 (streamRepeat 0)) (streamMap (+1) (interleaveStreams (streamMap (+1) ruler) (streamRepeat 0)))))
    S 0 (S 1 (S 0 (streamMap (+1) (interleaveStreams (streamMap (+1) ruler) (streamRepeat 0))) (streamRepeat 0)))

and so on and so forth. There's a lot of substitution going on, but notice the general pattern emerging: we obtain an expression of the form `S 0 (S 1 (S 0 (...)))` - the `ruler` series. Since it is of the form `S x (S y (S z (...)))` where `x`, `y`, `z` and so on are all literals, it can be lazily evaluated upto whatever point we need.

In the case when we type in `ruler` at the `ghci` prompt, as we did above, that point corresponds to 20 terms of the series.

##References

The course mentioned in the beginning of this post is Brent Yorgey's fantastic *Intro to Haskell* course. The course materials, including lecture notes and assignments are available online, [here](http://www.seas.upenn.edu/~cis194/spring13/).