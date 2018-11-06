---
layout: post
author: zhiayang
title:  "Post Rewrite Thoughts, and Other Stuff"
date:   2018-11-06 2230 +0800
categories: flax rewrite generics
---


The rewrite was "officially" completed some time ago -- I say "officially" because we really finished it some months ago -- with the release of v0.30.0.
There've also been some minor releases along the way, but most of them are just surface changes that don't affect the workings of the compiler.

Here's a quick run-through of some of the changes, for the full release notes see [the github page](https://github.com/flax-lang/flax/releases/tag/0.30.0).


### Unions

Tagged unions are now a thing! The semantics are like one would expect from tagged unions:

```rust
union option<T>
{
    some: T
    none
}

do {
    let x = option::some("foobar")

    using option as _
    let y = some(456)

    printf("x = %s, y = %d\n", x as some, y as some)
}
```

We currently don't have `match` (or `switch` -- haven't decided) statements yet, so for now we need to check each case with `if/is` (same deal with `any`).
However, there's also some neat stuff going on, like the type inference. We never needed to specify what exactly `T` was -- `x` is `option<str>` and `y`
is `option<int>`.

Also, we can `using` generic things, which allows us to use the variant name without needing to provide the polymorphic arguments.


### Polymorph Instantiation

Type arguments can now be positional -- meaning we don't need to do `option!<T: str>`, just `option!<str>`! The rules are exactly the same as that for
function calls, in that we cannot have positional arguments after named arguments. Also, notice the exclamation mark. I don't think we need it to parse
successfully, but I figured it was a good idea. Maybe not, we'll see.



### Internal Restructuring

Fundamental in our achievements was the large refactoring of the compiler's resolver system, both of generic parameters and of functions in general. To
support the levels of inference required to use generic unions non-invasively, quite a lot more information had to be passed around. I took the chance
to refactor the `TypecheckState` and moved a lot of the function-resolution and polymorphic-solver code into their own free-standing functions.

Hopefully the system is quite a bit more robust now.



### Syntax Changes and QoL Stuff

1. Most obvious of all, we've changed to differentiate between static access and instance access, just like in the old days of Flax. The former is done with
`::`, and the latter with `.` -- which should be no surprise to anyone.

2. Next, we've added the `$` shorthand to refer to `.length` in a subscript context, with inspiration from D.

3. What I like to call the `ffi-as` construct:

```rust
export gl
public ffi fn vertex(x: int, y: int) as "glVertex2i"
public ffi fn vertex(x: float, y: float) as "glVertex2f"
public ffi fn vertex(x: double, y: double) as "glVertex2d"
public ffi fn vertex(x: int, y: int, z: int) as "glVertex3i"
public ffi fn vertex(x: float, y: float, z: float) as "glVertex3f"
public ffi fn vertex(x: double, y: double, z: double) as "glVertex3d"

// ...
gl::vertex(1, 1) // calls glVertex2i
gl::vertex(1.0, 1.0, 0.5) // calls glVertex3d
```

Isn't that great? They overload and everything!

4. Chained comparison operators, like in Python or something:
```rust
if 3 < x < 10 => ...
```









