---
layout: post
author: zhiayang
title:  "Scoping Problems III"
date:   2017-12-01 2200 +0800
categories: flax problems
---

Some progress has been made, but some other progress has been lost. So, it turns out that scoping is somewhat important during code generation,
namely for the `ValueTree`s that we have. It would be simple to just rely on a program-wide `valueMap` to resolve values from definitions, but the
issue comes when we're doing reference counting.

Since automatic reference counting is entirely dependent on semantic scopes to know when to decrement the reference count, each list of reference
counted values must be intrinsically tied to a scope, and have its reference count be manipulated in accordance with that scope.

So, we have a dilemma here; reinstate the code-generation scoping system, or find another way to store the reference counted things. The reason why we're
facing the problem that we are, is that when we 'leave' the current scope to generate, for instance, our function-call-target, the value-tree doesn't change,
and we mess up the book-keeping of the reference counted things.

One possible solution is to tie the reference counted values with the SST nodes directly; each `Block` will store its own list of reference counted things,
instead of relying on the `CodegenState` to do it? Is this feasible? By the time we reach the code-generation state, there really should be no issue with this
thing. Each `Block` by right should only be touched once, and be generated once; so it should be ok to associate the `fir::Value`s with the node itself.


One problem I have with all of these 'hack-ish' solutions is that I have no idea how well they'll cooperatate with generic functions and types, and further
down the line proper incremental compilation. It feels like the entire compiler is built on the assumption that we have complete and unfettered access to
all nodes in all files, which might not be the case when we separate out compilation into proper modules and such.

Time will tell how it goes -- I really do not foresee the mental endurance or capacity for another rewrite when the time comes.


The best part is that this isn't even the initial issue I set out to solve; it was uncovered by debugging and setting up a reproducible test case. I haven't
gotten around to debugging the initial issue yet, which is that we're somehow not generating global variables properly, leading to a compilation failure
in the `intlimits` test.

I don't even have the slightest idea about that one.


#### Solved
Turns out the 'initial' problem was a manifestation of the deeper problem with `ValueTree`s; I've just eliminated their entire existence from the
compiler, and made `ControlFlowPoint`s store the reference counting things instead -- this way we don't need to modify the SST nodes to insert
stateful values, and the nesting nature is controlled by a simple stack that's run through during codegen instead of stored permanently somewhere
we don't need.
