# Limitations

Here is a non-exhaustive list of relevant issues and warnings to consider when using Glistix.

## Table of Contents

- [Overriding packages incompatible with Nix](#overriding-packages-incompatible-with-nix)
    - [Dealing with local package conflicts when patching](#dealing-with-local-package-conflicts-when-patching)
- [Missing tail-call optimization](#missing-tail-call-optimization)
- [Lazy evaluation](#lazy-evaluation)
    - [Discarded expressions](#discarded-expressions)

## Overriding packages incompatible with Nix

Many **existing Gleam packages** available over Hex **will not work on Nix** by default, as **they rely on FFI with Gleam's typical targets** (Erlang and JavaScript).
The workaround is to **create Nix-compatible forks** of those packages. To use them, however, we have the following **challenges:**

1. Currently, **Gleam does not have a proper requirement overriding system** (tracked at [upstream issue #2899](https://github.com/gleam-lang/gleam/issues/2899)). As such, we have to use those forks as **local or Git dependencies of each top-level project depending on them.** That way, **they override transitive Hex dependencies with the same name,** thus allowing the fork to be used even if the package appears as a transitive dependency (i.e. a dependency of a dependency, at any level).

2. However, **Gleam does not currently support Git dependencies** (tracked at [upstream issue #1338](https://github.com/gleam-lang/gleam/issues/1338)). As such, **we depend on local paths pointing to cloned Git submodules.**

3. Finally, **each fork needs to have the same name as the package it's patching** (due to a compiler requirement, as there is no built-in patching yet) and, as such, **cannot be published to Hex** (as it can't replace the patched package itself, of course).

It is expected that a proper solution to these problems will be available in the future. For now, the best we have is **cloning forks as Git submodules and using them as local dependencies.** Note that this also **requires adding those submodules as inputs to your project's `flake.nix`** and to the `submodules = [ ... ]` variable in the default `flake.nix`. This **ensures the submodules will be available when using the flake to build your project.**

However, doing that wouldn't let you publish your packages to Hex, since you cannot depend on local packages as a Hex package (you must depend on other Hex packages). Therefore, as a temporary workaround, **Glistix added a `[glistix.preview.hex-patch]` setting to `gleam.toml`**, where you can **override your package's dependencies once published to Hex.** This allows you to **replace a local dependency with a Hex dependency, but only when publishing.** For example:

```toml
# ...
[dependencies]
# While developing, depend on your local patch of the stdlib...
gleam_stdlib = { path = "./external/stdlib" }

[glistix.preview.hex-patch]
# However, once published to Hex, your package will depend
# on the regular stdlib instead, for compatibility with
# other packages.
gleam_stdlib = ">= 0.34.0 and < 2.0.0"
```

As such, **users of your package will be responsible for patching** by themselves, but we expect patching to only be truly necessary for core packages (e.g. `gleam_stdlib`), which most Glistix users will have to patch anyway (until we get proper requirement overriding).

Don't worry though - `glistix new` **automatically patches `gleam_stdlib` for you** by setting up `gleam.toml` and cloning it as a submodule to `external/stdlib`. However, you will have to do that by yourself for any new package patches you might need (e.g. [`glistix/json`](https://github.com/glistix/json)).

### Dealing with local package conflicts when patching

When you're patching a Gleam package with a Nix-compatible fork through Git submodules / local dependencies and **another local dependency transitively depends (locally) on that same package**, **you will get a conflict error** as you can't have two paths for the same local dependency. For the same reason as above, and given this is possibly a common problem when applying patches (maybe that fork of the package you're using depends on another fork you depend on, such as `gleam_stdlib`), **Glistix implements a temporary workaround** in the form of the **`local-overrides` setting of `[glistix.preview]`**. It is a list of packages (such as `["gleam_stdlib", "json"]`) for which **the local dependency specified by the top-level project should override any transitive local dependencies with the same name.** As such, the conflict is solved by using the path specified by your top-level project (the one being compiled). If the top-level project does not depend on that package, the conflict remains and `local-overrides` doesn't do much, so make sure to use `local-overrides` only for dependencies of your root project.

For example, if you clone both `glistix/stdlib` and `glistix/json` repositories as submodules to `external/stdlib` and `external/json` respectively, and depend on them on `gleam.toml` via `gleam_stdlib = { path = "./external/stdlib" }` and `gleam_json = { path = "./external/json" }` respectively, **you will get an error** because `gleam_json` depends on `gleam_stdlib` at `external/json/external/stdlib`, while you depend on `gleam_stdlib` at `external/stdlib`. The fix is to add
```toml
[glistix.preview]
local-overrides = ["gleam_stdlib"]
```
to your root project's `gleam.toml`, and Glistix will know how to solve the conflict (use `gleam_stdlib` from `external/stdlib`). Note that **this setting is ignored for non-root packages** (dependencies cannot apply `local-overrides`).


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
