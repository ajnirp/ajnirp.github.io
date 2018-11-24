---
layout: post
title: Rotating a Square Matrix
published: true
mathjax: true
---

How do you rotate a square matrix counter-clockwise? That's easy enough: you map the cell located at $(r, c)$ to the cell located at $(n-c-1, r)$, where $n$ is the length of a side of the square matrix.

Today I stumbled across another way to do this. The idea is to flatten the matrix out and derive a linear map between the 1D indices of the flattened matrix and the 1D indices of the flattened rotated matrix. The relation is $(ni + n - 1) mod (n^2 + 1)$. That is, given an index $i$ into the flattened version of the original matrix, the corresponding index in the flattened version of the rotated matrix will be $(ni + n - 1) mod (n^2 + 1)$.

The index $i$ is an integer between $0$ and $n^2-1$, because it is an index into the flattening of a square matrix of side $n$. In fact, $i = nr + c$. Intuitively, we make r jumps of stride n, and then offset ourselves by c to get to the cell located at $(r, c)$. Note that $r$ and $c$ are integers lying in $[0, n-1]$.

How do we derive the relation $(ni + n - 1) mod (n^2 + 1)$? By trial and error. My starting point was to take a 4x4 matrix and rotate it to the left, and to observe that there was an even spacing between the values of the 1D indices in the output - and that that spacing was equal to $n$ itself. It was natural to guess that there was a modulo $n^2$ involved, but that failed very quickly. I then tried $n^2 + 1$, and it worked!

Why does this work? The derivation is straightforward. We relate 2D coordinates in a square matrix to the index of the corresponding cell in the flattened version: $(r, c) => (n-c-1, r)$. We already know how to map 2D coordinates between matrices when one is a left-rotation of the other. So we just need to derive a relation between the 1D indices, and we're done.

$i\_1 = c + nr \\\
i\_2 = r + n(n-c-1) = r + n^2 - nc - n$

The second line above comes from the formula for mapping a 2D coordinate to a 1D one in the flattened array. Here the coordinate is $(n-c-1, r)$, which is where $(r, c)$ goes after we rotate the matrix.

$(ni_1 + n - 1) mod (n^2 + 1)\\\
= (nr + n^3 - n^2c - n^2 + n - 1) mod (n^2 + 1)\\\
= (nr - n^2c + (n - 1)(n^2 + 1)) mod (n^2 + 1)\\\
= (nr - n^2c) mod (n^2 + 1)\\\
= (nr - c(n^2 + 1) + c) mod (n^2 + 1)\\\
= nr + c$

...and we're done.