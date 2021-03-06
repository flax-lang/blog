---
layout: post
author: zhiayang
title:  "Import-as and Scoping Problems"
date:   2017-11-29 02:59:53 +0800
categories: flax problems
---

Here's some notes on how to fix this import-scope-problem, hopefully.

1. Functions need to be decoupled from their scopes more.

	Here's the thing: we perform module import shennanigans after each module is typechecked. So, the proper thing to do is that
	the 'scope' of each definition is its *original* scope. Any other way to refer to it is like a nickname, and doesn't change
	the original.

	However we do it, we must ensure that the SST nodes in the original typechecked module are left *untouched and immutable*, since
	we don't want module X that imports module Y to change how module Z sees module Y's definitions. (duh, you shouldn't be changing
	the state of something that you're importing since you don't own it)

	The real problem comes during code generation. For some reason (not-yet-looked-into), we need to 'teleport' to the original scope
	of the definition before continuing with code generating. Seems quite weird, honestly.

	In theory, if we implemented everything according to my internal mental model, during typechecking all references that may be involved
	in any kind of scoped operation should already have their target resolved (in terms of a `sst::Defn*` pointer) and stored. So, there
	really shouldn't be any kind of scope-related operation during code generation. Given that we're in fact not doing any kind of dot-op
	resolution at code-generation time, this should be doable.

	We need to investigate whether we can eliminate the necessity to have the scope-teleporting system in place. If so, it makes our job
	slightly (or a lot) easier. If it cannot be done, then appropriate changes must be made, once investigations show the exact reason
	we need the teleporting nonsense.


	After that, it shouldn't matter in what scope we generate the function.



2. Definitions shouldn't be explicitly aware of the scope in which they're being poked at.

	The current situation is that definitions are teleporting around and being self-aware. The thing we need to do is... not to have that.
	Assuming that point (1) was successfully resolved (ie. definitions care less about their scope), then during code-generation we shouldn't
	have to care *at all* about scopes.

	The current module being typechecked will, by guarantee, be importing modules that have already been typechecked, and their public
	definitions inserted into the proper `StateTree` location in the current module. Thus, when we traverse the StateTree to resolve
	an identifier (for example during a dot-op resolution), we get the definition -- if all goes according to plan, it doesn't care about
	its scope; we just set the target of the dot-op (for example) and hand it off to code generation.

	One thing -- during dot-op checking, we perform scope teleportation. However, at typecheck time this shoudn't be an issue, since whatever
	we're trying to actively resolve should only reside in our own module and whatever that's outside should have already been resolved.
	So, by right there shouldn't be any issues.

	The only reason we're teleporting around when resolving scopes is so we don't have to manually 'resolve' any things ourselves.
	For such a statement: `foo.bar()`, if we just teleport to the scope of `foo`, and let `bar()` typecheck itself, then we basically
	do what needs to be done without duplicating work.

	Sidenote: (TODO: FIXME:) one issue that I can forsee (again, based on my limited mental model that assumes 100% perfect implementation) is
	that, when typechecking the function arguments for the call to `bar()`, (eg. `bar(x, y, z)`), we will basically "absorb" whatever's in `foo`,
	such they will implicitly "prefer" `foo.x`, `foo.y`, and `foo.z` (if they exist). This can (will?) cause unexpected shadowing issues that are
	counter-intuitive. This is an issue.



3. StateTree aliases need to exist

	This is required for import-as, and probably using, statements. Again, if everything was implemented according to my internal mental
	model, then this should be as simple as twisting the `subtrees` map a little bit. Assuming that module `X` is aliased to the
	identifier `foo`, then `subtrees["foo"] = X`.

	For nested usings, eg. `using foo = X.Y.Z`, it might be possible to just `subtrees["foo"] = Z`. If it works, it greatly simplifies a great
	load of bullshit. Plus, it should following scoping rules:

	```rust
	do {
		using foo = bar;
	}

	foo.hello()		// <-- invalid
	```

	The using statement is placed in its own stree due to the 'do'-block pushing a new scope, and the 'fake-mapping' doesn't 'escape' from
	the scope.

	Note that there might need to be some considerations for 'public using', where we export an alias. Most of the time we don't want to be
	importing aliases (it's like putting 'using namespace std' in a header file, thus giving everybody including that header to contract
	metaphorical, programming-STDs), so we need a way to "filter" out these. Right now we can do a simple check consisting of (first, if the
	map-key matches the actual tree name, and if true, if the parent of the tree is the current parent)

	(note: the second check is for the case were we have `using foo = bar.foo`, where the names would match but the trees aren't directly
	related so it'd still be an alias)


#### TODO
However, I feel like this entire system is slightly somewhat kind of completely utterly not-very-convincing-nor-robust, so,
provided what we say *actually works*, we should, *at some point*, look into actually making some proper book-keeping for using/aliases.

Also a side issue is that, since named StateTrees are paired with NamespaceDefns, I think a similar thing needs to be done for the 'defns'
list that we have, and possibly the same, non-robust "is-this-an-alias" check consisting of name checking. Problem is, there's no "parent"
thing for us to do the second check.

Based on an entirely un-verified preliminary consideration, it might not be necessary to maintain `sst::NamespaceDefns`. In fact, they're
really unnecessary and feel like duplicates of the stuff that StateTrees contain, just inferior. If we remove them, then the alias-checking
problem won't be an issue.

On a surface-level kind of thing, it seems like resolutions of identifiers primarily use the `StateTree` definitions map to find their targets,
so it should be entirely feasible to eliminate `NamespaceDefn`s at the SST level.
