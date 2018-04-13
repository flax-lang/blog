---
layout: post
author: zhiayang
title:  "Class Constructors"
date:   2018-01-27 2200 +0800
categories: flax classes
---

Currently, we know that we need to call the proper constructor when making class values. Problem is, do we even want to allow implicit instantiation
of classes? We can enforce that class types must all be explicitly initialised. This would disallow the following pattern, however:

```rust
var foo: SomeClass
if(some_condition)
	foo = getClass(10)

else
	foo = getClass(20)
```
Even though a human can tell that `foo` will always be given a valid value regardless of the branch taken, the compiler in its current state (and indeed
even clang++, I believe), cannot.

For the current iteration, I think we cannot enforce the 'explicit initialisation' matter because the compiler simply doesn't allow facilities to support
productive programming *without* it. Once we can get some level of proper analysis about the thing above (which should be simple; loops don't guarantee
anything, and for if-elses: all `if` branches must explicitly assign to it, and there *must* be an `else` branch), we can probably do the original idea.

In that situation, we can leave the default value of `foo` as a `memset` of 0. We cannot use it before it is assigned so its actual contents don't matter
(but unlike C/C++ is actually in a deterministic state, just perhaps not valid to things that operate on the class); so we don't have to do unnecessary
work initialising a dummy value which would just be replaced eventually.

Note: use-before-init is also one of the analyses that we need to have before we can implement this.

