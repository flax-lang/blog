---
layout: post
author: zhiayang
title:  "Decomposition (aka Structured Bindings?)"
date:   2017-12-02 0800 +0800
categories: flax problems
---

Decomposition should be able to work like it did before the rewrite, with one small caveat -- we need to be generating fake definitions during the typecheck
phase, so that the `valueMap` has a unique referrer during the code generation phase. Otherwise, it should be good.

One case worth considering is when wanting to declare multiple variables to have the same value, like so:
```rust
let (a, b, c) = 30;
```

Currently that would be a compiler error, since `30` isn't a tuple type with 3 members. One potential solution is to use a 'splat' operator that 'multiplies' its operand an specified number of times, like so:

```rust
let (a, b, c) = ...30;
```

Since this isn't going to be code-generated, at the assignment site during typechecking we can just check if the RHS is one of these splat operators,
and do appropriate magic to handle it. Otherwise, it's going to be an invalid construct.

There's also the case where we want to splat a single value into an N-sized array containing N such values, but there's no way to do this at compile-time,
and will probably need to be a runtime operation. No big deal, not a high priority.



