# Nix backend

The **Nix codegen backend** is very much based on the JavaScript backend, and is implemented as follows in the compiler:

1. The entrypoint is at `compiler-core/src/nix.rs`. This Rust module **implements the compilation of a Gleam module (file) to a Nix file**. The `module` function is the entrypoint and invokes the `compile` method of `nix::Generator`, which is ultimately responsible for generating the module's corresponding Nix code.
    - `nix::module` is invoked at `compiler-core/src/codegen.rs` whenever the project is being compiled with the Nix target.
    - `nix::Generator` contains methods such as for generating a function definition, a record definition and module constants, as well as handling imports and exports, delegating to `nix::import` where necessary (see below).
2. The file `compiler-core/src/nix/expression.rs` is very important, as it implements the **compilation of each kind of Gleam expression to Nix code.** It contains a `Generator` struct which is **initialized once for each function** in a module, and it essentially traverses the Typed AST for each expression, converting each inner expression to Nix, recursively.
3. The file `compiler-core/src/nix/import.rs` handles imports, generating a series of `import` lines which appear at the top of the generated Nix file, as well as listing names to be exported at the bottom of the file (which are picked up by `nix::Generator::compile`).
4. The file `compiler-core/src/nix/pattern.rs` is responsible for **traversing patterns** (used in `case` clauses and `let` / `let assert` statements) and **converting them into assignments and conditionals** (in an abstract manner), which are then consumed by generators of `case` and `let/let assert` at `nix/expression.rs` and translated to `if...then...else if` statements where appropriate.
5. The file `compiler-core/src/nix/syntax.rs` contains multiple **helpers to generate specific kinds of Nix expressions.** For example, to generate a `let...in` expression, to generate an attribute set from a list of name/value pairs, and so on. These helpers are extensively used by other submodules of `nix`.
6. Tests are located in `compiler-core/src/nix/tests`.

This page is under construction.
