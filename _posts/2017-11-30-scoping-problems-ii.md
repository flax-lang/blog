---
layout: post
author: zhiayang
title:  "Scoping Problems II"
date:   2017-11-30 02:59:53 +0800
categories: flax problems
---

So, it turns out that the problem I was describing in (3) ([previous day]({{ site.baseurl }}/2017-11-29/import-as-and-scoping-problems)) turns out to be true. We teleport into the scope of the dot-operator
when doing typechecking:

```rust
let a = 10;
let b = 30;
foo.bar(a, b);
```

In the above snippet, `a` and `b` are resolved in the scope of `foo`, instead of the current scope. Thus, the two local variables `a` and `b` are not
seen at all, leading to a compilation error.

Because we need to know the type of the arguments (and hence typecheck them) in resolving overloads, we still need to be in the previous thing.
So, possibly the solution is to break apart our resolution into separate steps, and inevitably duplicate some code...

1. Teleport to the scope of `foo`, and find function candidates from there.
2. After getting the candidates, teleport back to the original scope, and perform overload resolution (and hence argument typechecking) there.
3. Then, it's done, hopefully.

This necessitates splitting up the resolution code so we can do more fine-grained resolving.
Another possible alternative is to typecheck each argument first, before teleporting to the function scope??? who knows.

