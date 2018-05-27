---
layout: post
author: zhiayang
title:  "Error Messages"
date:   2018-05-27 1400 +0800
categories: flax generics
---

While I'm procrastinating more important features of the compiler, I've decided to revamp the compiler error output facilities. A few commits ago, all
we really had was one kind of error -- print a message, show the context, draw some squiggles -- that's it. (though we've recently changed how the
context is printed as well)

```
error: No such function named 'bar'
at:    supertiny.flx:173:10
(call site):
    |
173 |    bar()
    |     ^

There were errors, compilation cannot continue
```

Now, we have much fancier errors:

```
error: No overload in call to 'printThings(number, [i64:])' amongst 3 candidates
at:    supertiny.flx:173:10
(call site):
    |
173 |    printThings(30, ...[ 1, 2, 3, 4, 5 ])
    |    ^^^^^^^^^^^

note: candidate 1 was defined here:
at:   supertiny.flx:118:5
    |
118 |    fn printThings(mul: f64, flts: [f64: ...])
    |                             ^^^^
                                  |> Mismatched type in parameter pack forwarding: expected element
                                  |> type of 'f64', but found 'i64' instead

note: candidate 2 was defined here:
at:   supertiny.flx:126:5
    |
126 |    fn printThings(mul: str, flts: [f64: ...])
    |                   ^^^       ^^^^
                        |         |> Mismatched type in parameter pack forwarding: expected element
                        |         |> type of 'f64', but found 'i64' instead
                        |
                        |> Mismatched argument type in argument 0: no valid cast from given type
                        |> 'number' to expected type 'str'

note: candidate 3 was defined here:
at:   supertiny.flx:134:5
    |
134 |    fn printThings()
    |       ^^^^^^^^^^^
            |> Mismatched number of arguments; expected 0, got 2 instead

There were errors, compilation cannot continue
```

Every site is only shown once, and the mismatched types are pointed out directly. It might feel slightly strange to have the squiggle pointing to the function
definition (because most frequently things are *called* wrongly), but it was a design decision -- it's far less cluttered, and honestly how does one properly
display the errors associated with each candidate at one call site context?

Behold, in true 16-colours:
![behold]({{ "/assets/error-message-screenshot.png" | absolute-url }})

Currently work is in progress to refactor the rest of our error reporting suite to be more robust and more configurable in terms of output.

