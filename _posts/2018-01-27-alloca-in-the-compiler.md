---
layout: post
author: zhiayang
title:  "Stack Allocations in Code Generation"
date:   2018-01-27 2100 +0800
categories: flax cleanup
---

In the current state of the compiler codebase, there's a lot less of the pattern where we create a temporary stack allocation, perform some operations,
then load to get the value -- compared to the previous iteration (pre-rewrite). This is because of the way LLVM's value types were defined, in not having
addressses of values.

However, there are still some potential cases where I think it can be slightly improved, such as calling class constructors. In that situation, the init
function needs a `self` pointer, but there isn't one (we're creating it!). The current solution is to create a temporary allocation (to get a pointer),
pass that to the constructor, then load it and return that value.

The problem is, is there a better way to do this?
