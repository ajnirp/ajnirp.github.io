+++
title = "Handshakes at a party"
date = 2025-11-16
draft = true
+++

Every month the Canadian Mathematical Society publishes [Crux Mathematicorum](https://cms.math.ca/publications/crux/), a collection of excellent math puzzles for secondary school and undergraduate students. In this post I'll discuss a fun problem from Vol. 51 No. 1 - **MA304**. You can read the issue online or download it [here](https://cms.math.ca/publications/crux/issue/?volume=51&issue=1).

## Problem statement

At a picnic, there are {{ katex(body="c") }} children, {{ katex(body="m") }} mothers, and {{ katex(body="f") }} fathers, with {{ katex(body="2 \le f \lt m \lt c") }}. Every person shakes hands with every other person. The sum of the number of handshakes amongst the children, amongst the mothers, and amongst the fathers is {{ katex(body="80") }}. How many persons attended the picnic?

## Sketching out a solution

To start with, let's restate the problem. We are told that

{% katex(block=true) %}
2 \le f \lt m \lt c
{% end %}

and also

{% katex(block=true) %}
\binom{c}{2} + \binom{m}{2} + \binom{f}{2} = 80 \\
\implies \frac{c(c-1)}{2} + \frac{m(m-1)}{2} + \frac{f(f-1)}{2} = 80 \\
\implies c(c-1)+m(m-1)+f(f-1)=160
{% end %}

where {{ katex(body="\binom{n}{m}") }} denotes the number of ways to choose a [combination](https://en.wikipedia.org/wiki/Combination) of {{ katex(body="m") }} objects from a set of {{ katex(body="n") }} distinct objects.

Can we bound our search space? If {{ katex(body="f") }} were too large, then {{ katex(body="m") }} and {{ katex(body="c") }} would be even larger and the {{ katex(body="c(c-1)+m(m-1)+f(f-1)) }} would exceed 80. In particular, with {{ katex(body="f = 7") }}, the minimum values of {{ katex(body="m") }} and {{ katex(body="c") }} are 8 and 9, which means {{ katex(body="c(c-1)+m(m-1)+f(f-1) = 7.6 + 8.7 + 9.8 = 170 > 160") }}. So {{ katex(body="f") }} is at most 6.