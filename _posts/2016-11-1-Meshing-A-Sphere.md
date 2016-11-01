---
layout: post
title: Meshing A Sphere
published: false
---

How do you mesh a sphere with triangles?

### Problem Statement

Let's be more precise here: suppose I give you a unique sphere by telling you the 3D Euler coordinates of its center, and its radius. Your job is to give me a list of triangles that together form a triangular mesh for the surface of this sphere. What algorithm will you use to do so?

Well, this is a trick question. You can't tile a perfect sphere with triangles. Doing so would mean that for each triangle in the mesh, there is a point somewhere in space that is equidistant from every point in the triangle's interior. Triangles are flat, so no such point exists. So let's ask a tougher question. How can you generate a triangular mesh that is *not* a perfect sphere but resembles one? Ideally, whatever algorithm you would use to generate such a mesh would have (one or maybe more) parameters that control the fine-ness of the mesh, and the more fine the mesh, the more it should resemble a sphere.

### A Simplification

First, let's reduce the problem statement a little. The fact that the sphere can be anywhere in space and have any radius does not really matter to our algorithm. If we solve the problem for a unit sphere centred at the origin, then we can simply scale and translate the output mesh's vertices by the appropriate amounts and we are done. So now our problem statement is slightly simpler. Our sphere is centred at the origin and its radius is 1.

### Iterative Refinement

While searching around, I found an elegant algorithm that does exactly this. The basic idea is conceptually very simple - we start out with an octahedron mesh, which has a known (and simple) geometry. This is what an octahedron looks like. It's basically eight equilateral triangles joined at their sides.

TODO: add picture of an octahedron

This looks nothing like a sphere just yet, but we can change that. What we have to do is **iteratively refine** the octahedron, and stop after a certain number of iterations. Refinement in this context means taking each triangular face of the current mesh and splitting it up into four triangles. Then, we scale each one of the coordinates of each one of the new triangles so that they are unit vectors. Once this procedure has been applied to each face of the current geometry, we count it as a single iteration. After a few iterations, we stop. The exact number of iterations we use is the parameter we alluded to earlier - a higher number of iterations means smaller triangles and a greater resemblance to a sphere.

This picture explains what's going on. Notice how the triangle bulges outward by a little amount on each iteration! Slowly, it begins to resemble a quarter section of a hemisphere. Eight such triangles together comprised our original octahedron. Over time, we end up something that looks a lot like a perfect sphere.

TODO: add picture here

### Motivation

Where did this problem come from? For an assignment in a modeling and simulation course I was taking, I had to  physically simulate collision handling. The collision detection library I was using expected input in the form of a *triangle soup* - a list of triangles that meshed a particular object. This meant having to model each object in the scene with a triangular mesh. For a cube, this is fairly straightforward. But for a sphere, I had to seek out a good algorithm, which motivated the subject of this blog post!