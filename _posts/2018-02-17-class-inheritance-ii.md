---
layout: post
author: zhiayang
title:  "Class Inheritance II"
date:   2018-02-17 2100 +0800
categories: flax classes
---

Nothing much has actually been done the past 3 weeks, from the time when the previous thing was written, other than the fact that we managed to get
`using` working on both namespaces and enum cases.

Apart from that, there are some issues that need to be resolved; firstly, the problem of accesssing base-class fields from a derived class without an
explicit `super.foo` (same thing with methods). For this, it should be somewhat straightforward (famous last words), because we don't allow overriding of
methods and fields. (kind of -- non-virtual overriding for methods is prohibited). Thus, if we first check for duplicates, we should be able to throw
appropriate error messages, and then just import the stuff in the base class to our own class. Assuming the base class has been typechecked prior (which
we enforce), then we should transitively also import any grandparent, great-grandparent, etc definitions into ourselves.

The next issue is constructors (or init-functions). For these, we want to only be able to call them with `super(...)`, instead of some other kind of thing.
Thus, when we import stuff into ourselves as above, we must exclude importing constructors.

The conflict is that we *should* be able to "hide" constructors in a derived class that's -- the opposite of what we want for methods. Reason being that we
always know the type of what we're constructing, regardless of the dynamic type; the same cannot be said of method calls.

So, base-class init methods must only be accessible from a `super()` call, and `init()` calls must only expose the derived class constructors.

