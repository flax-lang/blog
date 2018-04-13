---
layout: post
author: zhiayang
title:  "Class Inheritance"
date:   2018-01-28 2100 +0800
categories: flax classes
---

As of this writing, we have a very basic version of inheritance working, that hopefully isn't too poorly architected such that we'd have troubles down the
line. So, as to how it works:

The actual mechanism of 'copying' base-class fields into the derived class happens at the *FIR* layer, and the front/middle-end code doesn't need to know
about the specifics of that. To the typechecker/code-generator, once we `setBaseClass(...)` on a `fir::ClassType`, the fields from the base class 'magically'
appear in the derived class, and we can access them by name like normal fields.

This naturally precludes us from having a field with the same name as another in the base class. We do this check recursively during typechecking, so you
cannot 'hide' any fields (and in the future, methods) of any class along your inheritance hierarchy.

Back to the FIR implementation: we don't actually *physically copy* the fields into the member list of the ClassType (because they're not *really* there).
Instead, when you call one of the field-related functions like `getElementIndex()` and `getElementWithName()` or something, if they're not found in the
current class, we traverse upwards till we find it (or error out). Note that, thus, `getElementCount()` returns the number of elements *defined in the class*,
*excluding* those defined in base classes.

In the translator (to LLVM), that's where we physically insert the fields from base classes into the data representation of the derived class. Theoretically,
it shouldn't be that hard to offset all accesses by 1 to have our vtable be at the beginning of the class (in memory). Theoretically.


Either way, there's still some gaps:

1. We can't do the implicit self thing when accessing base-class fields -- right now we need to use `self.base_field` to access it. Will be fixed.
2. Methods are completely not handled at all, and we've got nothing on the front of dynamic dispatch at all. Oh well.
3. Inheriting from a non-terminal class (eg. `A : B`, where `B : C`) has not been tested. Although it should work, I have no idea.
