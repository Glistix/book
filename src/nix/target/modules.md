# Modules

In Gleam, each module corresponds to a single `.gleam` file, and has its own exported names (`pub`), which includes functions, constants and type constructors.

When translated to Nix, **each Gleam module becomes a single Nix file.**
For example, if your package is named `hello`, the file at `src/my/module.gleam` will generate a file `build/dev/nix/hello/my/module.nix` on build.

Each module file **evaluates to an attribute set containing all exported names.**

For example, the module below:

```gleam
pub type Type {
  Constructor(a: Int, b: Int)
}

pub const constant: Int = 5

pub fn name() {
  "Hello World!"
}
```

transpiles to

```nix
let
  Constructor = a: b: { __gleamTag = "Constructor"; inherit a b; };

  name = { }: "Hello World!";

  constant = 5;
in
{ inherit Constructor name constant; }
```

## Prelude

The prelude is always at `build/dev/nix/prelude.nix` by default, and contains functions which are automatically imported by the compiler as needed.
