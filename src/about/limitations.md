# Limitations

Here is a non-exhaustive list of relevant issues and warnings to consider when using Glistix.

## Table of Contents

- [Lack of requirement overriding and Git dependencies](#lack-of-requirement-overriding-and-git-dependencies)
- [Missing tail-call optimization](#missing-tail-call-optimization)
- [Lazy evaluation](#lazy-evaluation)
    - [Discarded expressions](#discarded-expressions)

## Lack of requirement overriding and Git dependencies

The Gleam ecosystem, whose packages are mostly available through [Hex](https://hex.pm), contains **many useful packages which can also be used with Glistix.** However, oftentimes you will find **packages which do not work on Nix,** because they **rely on FFI with the usual Gleam targets** (Erlang and JavaScript). This includes, most importantly, **Gleam's standard library**. The way to deal with this is to **use forks of those packages patched for Nix support.**

However, Gleam does not have a requirement overriding system yet (tracked at [upstream issue #2899](https://github.com/gleam-lang/gleam/issues/2899), so we **depend on local and Git dependencies to those forks**, as **those kinds of dependencies have priority over transitive Hex dependencies** (so we get some initial patching support that way). Additionally, however, **Git dependencies aren't natively supported** (tracked at [upstream issue #1338](https://github.com/gleam-lang/gleam/issues/1338)), so we have to use **local dependencies to Git submodules** in order to use patches hosted in external Git repositories.

For more information, including **steps to override a package**, please check the page on ["Overriding incompatible packages"](../recipes/overriding-packages.md).

## Missing tail-call optimization

Compared to Gleam's Erlang and JavaScript targets, Glistix's Nix target **does not have tail-call optimization.** This means that **recursion continually grows the stack** and, as such, **it is possible to trigger stack overflows** on Nix through deep enough recursion. As a consequence, at least for now, **you should avoid relying on recursion to process very large data.**

There are some ways to try to work around this limitation:

1. Gleam's built-in `List` type works as a linked list across all targets (including Nix). It is therefore **ideal for recursion**, but **should be avoided when recursion needs to be avoided** (e.g. because the list might have thousands of elements). In those cases, **consider using Nix's arrays** (called "lists" in Nix's official docs) instead. The [`glistix_nix` package](https://github.com/glistix/nix) contains **various helpful functions** to aid you in using arrays. The array functions avoid recursion where possible (functions using recursion indicate they are doing so clearly in the docs - they are few).

2. It is possible to use Nix's `builtins.genericClosure` function to improve efficiency when traversing lots of data, as it **allows creating arrays from existing (possibly recursive) structures**, but **is not recursive** itself, thus allowing for some light "tail-call optimization" in some cases, particularly when you need to create arrays.
    - For more information, check out this great NixOS Discourse thread by user _sternenseemann_: [https://discourse.nixos.org/t/tail-call-optimization-in-nix-today/17763](https://discourse.nixos.org/t/tail-call-optimization-in-nix-today/17763)

## Lazy evaluation

Nix code is **evaluated lazily**, meaning **an expression isn't evaluated until requested.** This has implications on side-effects, such as when using logging for debugging during development; more importantly, however, **values which aren't evaluated lead to the creation of thunks** in memory; Creating too many thunks **can lead to slowdown and/or high RAM usage.** Therefore, and especially when working with complex and/or heavy programs or code within Nix, **try to reduce recursion and memory usage as much as possible** to avoid surprises.

### Discarded expressions

Due to the above, it is natural that expressions not bound to any variables would never be evaluated (in principle), due to laziness. To tackle that, **Glistix ensures all discarded expressions are evaluated** (at least shallowly). For example:

```gleam
pub fn main() {
  // The panic below won't run due to laziness.
  // The variable is never used.
  let var = panic as "message"

  // The panic below, however, WILL run.
  // Glistix forces discarded expressions
  // to be evaluated before the function's
  // return value using Nix's `builtins.seq`
  // functionality.
  panic as "other message"

  // The panic below will also run.
  let _ = panic

  // However, the panic below won't run,
  // as the evaluation of discarded expressions
  // is shallow.
  // This might change in the future.
  #(panic as "inside tuple")

  // It is worth saying that any returned values
  // are evaluated, of course.
  Nil
}
```

To force deep evaluation of expressions, you can use the `nix.deep_eval` function from [the `glistix_nix` package](https://github.com/glistix/nix):

```gleam
import glistix/nix

pub fn main() {
    nix.deep_eval(#(panic as "this will run"))

    Nil
}
```

Additionally, **assertions are always evaluated:**

```gleam
pub fn main() {
    // This will cause a panic, even though
    // this expression doesn't bind any variables.
    let assert False = True

    Ok("finished")
}
```

Glistix does this as certain packages rely on this behavior, such as `gleeunit` (the default test runner), which generally **encourages performing multiple assertions in one function**, for example - however, said assertions are usually done through **function calls not bound to any variables** (or `let assert` expressions). In other words, if discarded expressions were ignored, **the test suites of multiple packages would simply not do (almost) anything,** as multiple assertions per test function are often used and their side effects (of `panic`king upon failure) relied upon.
