---
layout: post
author: zhiayang
title:  "Virtual Methods"
date:   2018-01-25 2100 +0800
categories: flax future classes
---

For starters, I think it should be necessary to mark functions as `virtual` at the declaration site *at the base class*, in order to allow dynamic
dispatch on that function. The following, non-exhaustive scenario list applies:

### A
We define a virtual method in the base class, and overload it in derived classes explicitly with an `override` keyword (or something similar).
It would be an error to virtually-override a method without such a keyword.

### B
We define a normal, non-virtual method in the base class, and attempt to overload it in a derived class. Without an explicit overload
keyword, this is an error -- we don't really want to support method-"hiding" the way C++ does, as it's most likely an unintentional thing.

### C
We define a non-virtual method in the base class, and attempt to overload it with the keyword, or by declaring another method but marking
it as `virtual`. This will create an error, since the base class did not indicate that the given method was supposed to be accessible via dynamic
dispatch.

### D
We define an extension for the base class, and using some as-yet-undecided syntax, re-declare a given method as a virtual function. Now, derived
classes see the original method as if it had always been declared virtual, and dynamic dispatch works. While this seems like (and in reality *is*)
extra work on everybody's part, it's much more explicit that we're re-jiggering something in the base class, and modifying it in such a way that the
original author might not have intended it to be done.


This might seem like some kind of futile discussion if we can just go into the library itself and edit the definition to make the function virtual, but
someday when we support some kind of binary format for libraries (and no longer do source-only kinda thing), it would come in handy.

However, we should consider the ramifications of modifying the ABI of a function like that. It would be fine-*ish* if we modified the ABI of such a method
if the changes were only visible to the current program -- since we compile the whole program simultaneously (well you know what I mean, in 1 TU), then
any and all extensions that modify classes like this would be visible to the rest of the program, and the compiler can adjust how it calls the given method.

On the other hand, if we're creating some kind of library, then it is impractical to either (a) make the ABI changes only affect the current program, or
(b) transparently allow other programs transitively depending on the base class to know about the new calling ABI.


At the end of the day, while we want safety, we cannot compromise *that much* on flexibility. If the original base class author did not mark the function
as virtual, it's a question of how much we want to 'bend' to allow users of that class to call a given method dynamically. It might have been an accidental
omission, or it might've been fully intentional.

Something like the `final` keyword could be used (but of course not that itself -- I think the well-known meaning of `final` should stay, ie. to mark
that a class cannot be derived from), to say that all methods defined in such a marked class cannot be "re-defined" later to be virtual, or perhaps
the opposite (that any and all given methods can be re-declared to be virtual in derived classes). Maybe an `open class { ... }`?








