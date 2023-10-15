---
title: "Vectorspace_vs_lattice"
date: 2023-09-21T10:14:39-07:00
math: true
---
So given a collection of linearly independent
vectors

$$v_1, v_2, \ldots, v_n$$

we can form the n dimensional vector space
with the above collection of $v_i$ as our
basis. In fact, we basically define
the dimension of a Vector space by the size
of the smallest set of vectors that span
the space. Importantly,
given *any* n linearly independent set
of vectors, you can regenerate the entire
space. If you have n+1 vectors, you
*know* that you can get an equivalent
n vectors from that set of vectors.

This is *not* the case with Lattices! To see
why, let's focus on the 1 dimensional case.

The vector (1) will span a lattice of all
integers. The lattice spanned by (2) or (3) will not.
However, you can recover the original lattice
with the span (2) and (3)! You can 