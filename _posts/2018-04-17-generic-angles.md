---
layout: post
author: zhiayang
title:  "Generic Angle Brackets"
date:   2018-04-17 2230 +0800
categories: flax generics
---

For most imperative programming languages, the issue of parsing generics is quite a headache, especially if one goes with the *de-facto* syntax
of using angle brackets `<` and `>` after an identifier to give the template arguments. This is a problem in Rust, C++, and Swift, to name a few.

The issue arises in situations like these:

```rust
someMethod(foo<A, B>(K))
```

Without knowing the kind of entities that `A`, `B` and `K` are, we cannot tell how it should be parsed:
```rust
// two boolean parameters to the method, comparing foo with 'A' and 'B' with 'K'
someMethod(foo < A, B > K)  // (K) can just be parsed as 'K', as can (((K)))

// one parameter to the method, the type of which is the result of a generic
// function call to the function 'foo' with type parameters 'A' and 'B', and
// an argument 'K'
someMethod(foo<A, B>(K))
```

I'm not sure how C++ handles it (most search results only talk about the change in C++11 which allowed `>>` to be used properly in templates), but GCC
might just use its symbol table in the parser to assist, but IIRC clang has a separate semantic analysis parse which frees its parser from having to
distinguish between types and names.

For C# and Swift, the parsers perform a *speculative parse* that attempts to parse a generic type list; if it fails, then it rewinds the token stream and
then parses it as a less-than operator. If it succeeds, it additionally looks at the token immediately following the closing `>`: if it is one of `(`, `)`,
`]`, `:`, `;`, `,`, `.`, `?`, `==`, or `!=` (Swift additionally allows `{`, `}` and `[`, but enforces that `[`, `.` and `(` must appear immediately after
the `>` with no intermediate whitespace), then it is parsed as a generic entity.

For Flax, we've also gone with the speculative parsing approach, but dispensed with checking the trailing token. Due to the fact that we enforce providing a
*mapping* for the type arguments, like so: `Foo<T: int>`, the presence of the `:` token means that we can discard the less-than operator case. it remains to
be seen whether or not we require checking the trailing token. In fact, it remains to be seen whether we want to enforce exposing the type parameter name
to the user at all, or just go with the usual `<T, K>` approach.

Finally, back to the `>>` issue -- due to the way our parser is structured, there *shouldn't* be any ambiguity when we refuse to parse `>>` as a token (and
for consistency, refuse to parse `<<` as well). When we're parsing a binary expression and are looking for a token's precedence, the lookup function can
peek ahead in the token stream if it finds a `>` or `<`, see if it's another `>` or `<` respectively, and return the appropriate precedence for `>>` and `<<`.
We also handle `<<=` and `>>=` by looking for a trailing `<=` and `>=` respectively.
