# Standard Library

The Gleam standard library can be used within Glistix through **Glistix's `glistix_stdlib` port** (developed at [https://github.com/Glistix/stdlib](https://github.com/Glistix/stdlib)).

It must be used as a patch over `gleam_stdlib` through the `[glistix.preview.patch]` configuration section. See ["Overriding incompatible packages"](../../recipes/overriding-packages.md) for instructions.

In the specific case of the standard library, however, **this is already done automatically** by `glistix new`.
