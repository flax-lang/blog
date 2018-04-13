---
layout: post
author: zhiayang
title:  "Mutability II"
date:   2018-04-14 0400 +0800
categories: flax mutability
---

So we finished implementing the whole 'enhanced-mutability-scheme' in the past week, basically exactly as described in the previous rant, using the Rust-style
`&T` and `&mut T` for pointers to immutable and mutable `T`s respectively. One difference is that instead of using the exact same syntax for doing address-of
as well, we've gone with always yielding an immutable pointer when using `&`. To get the mutable variant, we use `foo as mut`, and `foo as !mut` for the
inverse. I think this will work out fine.

Next, we defined which kinds of things give mutable pointers; dynamic arrays (`[T]`) are always mutable, as are `string`s. For slices, their mutability depends
on what they're sliced from -- they'll be mutable for the two aforementioned types, immutable for all other types, and will inherit their mutability if they
come from a slice themselves.

It's pretty straightforward, and we've got a nifty syntax to specify that a slice type should be mutable (this changes the mutability of the data pointer):
```rust
let x: [mut int:]
let y: [int:]
```

Finally, we changed string literals `"Hello there"` to have slice type (`[char:]` to be precise), to be more in-line with the new premise of `string` -- which
is to serve as the mutable string class (a-la `std::string`). We've also added an alias in the form `str` to refer to `[char:]` more ergonomically. Of course,
implicit casting rules have been updated as well:

```rust
let x = "Hello there"       // type = [char:]
let y = "General Kenobi!"

// implicit casting to &i8 when required, similar to full strings
printf("%s. %s\n", x, y)

fn add_to_collection(fine_addition: str) { /* ... */ }

// string will implicitly cast to str, but not vice-versa
add_to_collection(string("this"))
```

Note that the constructor syntax exhibited above (`string("this")`) doesn't actually work. *Yet*.
