---
layout: post
author: zhiayang
title:  "Mutability (and a little Generic Types)"
date:   2018-02-17 2100 +0800
categories: flax mutability generics
---

So, managed to get generic structs (and presumably classes and all that jazz) working on a basic level. Not sure what else is missing. I'm personally quite surprised that it didn't take a gargantuan amount of effort to get it working -- possibly because I didn't try to start my endeavour by writing a 1k-line parametric constraint solver to begin with?

Anyway, one ergonomic issue has to do with the mutability of pointers; when we have something like this:
```cpp
int* foo = ...
foo = X         // ok
*foo = Y        // ok also

const int* bar = ...        // same as int const* bar
bar = A         // ok
*bar = B        // not ok

int* const qux = ...
qux = C         // not ok
*qux = D        // ok

const int* const wob = ...
wob = I         // not ok
*wob = J        // not ok
```

Right now, in Flax we can only express either const pointer to const memory, or variable pointer to variable memory. We need a way to express the two
in-between scenarios, namely variable pointer to constant memory, and constant pointer to variable memory.

I'll just borrow the Rust syntax, since it seems to work well especially with our new use of `&` for pointers. The mutability of the pointer itself will be
determined by whether `let` or `var` is used (see: `let` and `let mut` in Rust), while the mutability of the pointer will be determined by the presence of
`mut` after the ampersand: `&mut T` versus `&T`. So, we would have these:

```cpp
int* foo                // --  var foo: &mut T
const int* foo          // --  var foo: &T
int* const foo          // --  let foo: &mut T
const int* const foo    // --  let foo: &T
```

Seems like a decent solution. We could always change the default (ie. use `const` or `mut`) trivially once we implement this thing.
