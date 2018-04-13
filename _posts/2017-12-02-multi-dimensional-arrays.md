---
layout: post
author: zhiayang
title:  "Multidimensional Arrays"
date:   2017-12-02 2300 +0800
categories: flax future
---

In the old system, we had an `alloc[N][M]` syntax that would give so-called 'multi-dimensional' arrays, which were really just an array of arrays.
Right now, I've removed the multi-dim alloc, but we want something to replace it, potentially for matrix purposes.

So, with the current syntax `alloc(T, N, ...)`, if you give more than one dimension, then you get a multi-dimensional array back. Of course, the number
of dimensions is fixed at compile-time, since we know how many lengths were passed to `alloc()`. The length of each dimension would be a runtime
variable, but fixed -- so you could have an NxM matrix where N and M are user-inputs, but you can only ever fix it as NxM. I feel like I explained that
very poorly.

Here's the in-memory representation of such a `MultiDimArray` with `K` dimensions of type `T`:

```rust
struct __multi_dim_array_K_T
{
	ptr: T* = ...
	dims: i64[K] = [ N, M, ... ]
}
```

The type of this array is `T[,,]`, for `K = 3`. The slight issue with this is that, for instance, if we are passing such a multi-dim array around, then
the size of each dimension is not fixed, even though the number of dimensions is; eg. we could be passing a 2x2 matrix to a function expecting a 3x3 matrix.
Both are 2D arrays, but one is clearly larger than the other.

To mitigate this problem, we're probably also going to have fixed-dimension arrays, of the type `T[N, M]`, which function like the fixed arrays of today,
where `T[2, 2]` is a distinct type from `T[3, 3]`. Otherwise, they'd function similarly and perform similar actions. One would be able to pass a fixed
multi-array to a variable-multi-array, but not vice-versa -- for obivous reasons.

However, there should be no confusion as to the variable nature of the non-fixed multi-dim array; they vary in the size of each dimension, yes, but only
*once* -- they are runtime-allocated, non-expanding arrays. So, if you did `let mda = alloc(int, 3, 3)`, then `mda` is a 3x3 matrix, but there is no way
to *expand* it to a 4x4 matrix for example, without first freeing, then re-alllocating `mda`.

Speaking of which, the members of these multi-dim arrays are stored in contiguous memory, and some math is done to access them. Since we know the number
of dimensions, the there should be no need for any loops at runtime.


