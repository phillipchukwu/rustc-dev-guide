This file offers some tips on the coding conventions for rustc.  This
chapter covers [formatting](#f), [coding for correctness](#cc),
[using crates from crates.io](#cio), and some tips on
[structuring your PR for easy review](#er).

<a name=f>

# Formatting and the tidy script

rustc is slowly moving towards the [Rust standard coding style][fmt];
at the moment, however, it follows a rather more *chaotic* style.  We
do have some mandatory formatting conventions, which are automatically
enforced by a script we affectionately[^not_true] call the "tidy"
script.  The tidy script runs automatically when you do `./x.py test`.

[^not_true]: Secretly, I hate tidy. -nmatsakis
[fmt]: https://github.com/rust-lang-nursery/fmt-rfcs

<a name=copyright>

### Copyright notice

All files must begin with the following copyright notice:

```
// Copyright 2012-2013 The Rust Project Developers. See the COPYRIGHT
// file at the top-level directory of this distribution and at
// http://rust-lang.org/COPYRIGHT.
//
// Licensed under the Apache License, Version 2.0 <LICENSE-APACHE or
// http://www.apache.org/licenses/LICENSE-2.0> or the MIT license
// <LICENSE-MIT or http://opensource.org/licenses/MIT>, at your
// option. This file may not be copied, modified, or distributed
// except according to those terms.
```

The year at the top is not meaningful: copyright protections are in
fact automatic from the moment of authorship. We do not typically edit
the years on existing files. When creating a new file, you can use the
current year if you like, but you don't have to.

## Line length

Lines should be at most 100 characters. It's even better if you can
keep things to 80.

**Ignoring the line length limit.** Sometimes -- in particular for
tests -- it can be necessary to exempt yourself from this limit. In
that case, you can add a comment towards the top of the file (after
the copyright notice) like so:

```
// ignore-tidy-linelength
```

## Tabs vs spaces

Prefer 4-space indent.

<a name=cc>

# Coding for correctness

Beyond formatting, there are a few other tips that are worth
following. 

## Prefer exhaustive matches

Using `_` in a match is convenient, but it means that when new
variants are added to the enum, they may not get handled correctly.
Ask yourself: if a new variant were added to this enum, what's the
chance that it would want to use the `_` code, versus having some
other treatment?  Unless the answer is "low", then prefer an
exhaustive match. (The same advice applies to `if let` and `while
let`, which are effectively tests for a single variant.)

## Use "TODO" comments for things you don't want to forget

As a useful tool to yourself, you can insert a `// TODO` comment
for something that you want to get back to before you land your PR:

```rust,ignore
fn do_something() {
    if something_else {
        unimplemented!(): // TODO write this
    }
}
```

The tidy script will report an error for a `// TODO` comment, so this
code would not be able to land until the TODO is fixed (or removed).

This can also be useful in a PR as a way to signal from one commit that you are
leaving a bug that a later commit will fix:

```rust,ignore
if foo {
    return true; // TODO wrong, but will be fixed in a later commit
}
```

<a name=cio>

# Using crates from crates.io

It is allowed to use crates from crates.io, though external
dependencies should not be added gratuitously. All such crates must
have a suitably permissive license. There is an automatic check which
inspects the Cargo metadata to ensure this.

<a name=er>

# How to structure your PR

How you prepare the commits in your PR can make a big difference for the reviewer.
Here are some tips.

**Isolate "pure refactorings" into their own commit.** For example, if
you rename a method, then put that rename into its own commit, along
with the renames of all the uses.

**More commits is usually better.** If you are doing a large change,
it's almost always better to break it up into smaller steps that can
be independently understood. The one thing to be aware of is that if
you introduce some code following one strategy, then change it
dramatically (versus adding to it) in a later commit, that
'back-and-forth' can be confusing.

**If you run rustfmt and the file was not already formatted, isolate
that into its own commit.** This is really the same as the previous
rule, but it's worth highlighting. It's ok to rustfmt files, but since
we do not currently run rustfmt all the time, that can introduce a lot
of noise into your commit. Please isolate that into its own
commit. This also makes rebases a lot less painful, since rustfmt
tends to cause a lot of merge conflicts, and having those isolated
into their own commit makes htem easier to resolve.

**No merges.** We do not allow merge commits into our history, other
than those by bors. If you get a merge conflict, rebase instead via a
command like `git rebase -i rust-lang/master` (presuming you use the
name `rust-lang` for your remote).

**Individual commits do not have to build (but it's nice).** We do not
require that every intermediate commit successfully builds -- we only
expect to be able to bisect at a PR level. However, if you *can* make
individual commits build, that is always helpful.
