# Modules

In Gleam, each module corresponds to a single `.gleam` file, and has its own exported names (`pub`), which includes functions, constants and type constructors.

When translated to Nix, **each Gleam module becomes a single Nix file.**
For example, if your package is named `hello`, the file at `src/my/module.gleam` will generate a file `build/dev/nix/hello/my/module.nix` on build.

Each module file **evaluates to an attribute set containing all exported names.** For example, a module with `pub type Type { Constr(a: Int, b: Int) }`, `pub fn fun()` and `pub const name: Int = 5` will evaluate to an attribute set with attributes `Constr`, `fun` and `name`.

## Prelude

The prelude is always at `build/dev/nix/prelude.nix` by default, and contains functions which are automatically imported by the compiler as needed.
