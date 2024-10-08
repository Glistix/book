# General modifications

Glistix has applied several patches and additions to the base Gleam compiler. We intend to document most of them here.

- The most important change is the addition of a **Nix compilation target**. This is reflected as a variant in the `Target` enumeration in `compiler-core/src/build.rs`.
This lead to changes across many files in the compiler, mostly related to replicating code otherwise meant for other targets for Nix as well.
    - We **created a Nix codegen backend** (which compiles Gleam code to Nix) for the Nix target at `compiler-core/src/nix.rs` and its submodules. There is some information about the Nix backend in the relevant section.
        - This required creating a `Nix` structure in `compiler-core/src/codegen.rs`, whose methods manage the Nix codegen process, compiling every module in a project.
        - This also required the creation of a `prelude.nix` file under `compiler-core/templates` to be consumed by the Nix backend.
    - We **updated the parser** (at `compiler-core/src/parser.rs`) to add support for the `@external(nix, ..., ...)` attribute.
        - This introduced new `external_nix`-like fields in multiple structures across the compiler.
        - This required changes to `analyse.rs` in order to validate the paths and names of external Nix functions.
        - Similarly, `format.rs` was also changed so that `@external(nix, ..., ...)` is properly kept after formatting.
    - This required a few changes in the **compiler's Cap'n Proto schema** (`compiler-core/schema.capnp` and the generated files at `compiler-core/src/generated`) in order to store information for the Nix target in cache (in particular, external Nix functions).
    - **Many compiler tests and test snapshots had to be updated** as a consequence of that and other changes.

- We have **customized several bits of the compiler** so that they display information relevant to Glistix instead of Gleam compiler. Those changes are **mostly minor,** and include changing error messages to point to the Glistix repository, for example.

- In addition to the point above, **the compiler's crates were renamed** to `glistix-*` instead of `gleam-*` and **their versions were changed** to Glistix's own version scheme (with the initial version being `v0.1.0`) to better reflect Glistix as a separate project.

    - As a consequence, the file `compiler-core/src/version.rs` was changed to reflect not the Glistix version (defined in `Cargo.toml` files), but the **Gleam compiler version we are basing ourselves on**. This means that **this file must be updated each time Glistix is updated to a new Gleam version.** That version is checked by packages to ensure they are being used with a compatible Gleam compiler, so using Glistix's own version would cause wrong results and/or incompatibilities when checking compiler version restrictions.

    - This also required **renaming all test snapshots** as they are prefixed with the crate name they apply to.

- We have **customized several CLI commands** to add Nix-specific functionality. This includes at least `glistix new`, `glistix build`, `glistix run` and `glistix test`. Please check the dedicated chapter for CLI more information.

- We have added a `[glistix]` section to `gleam.toml` for Glistix-specific configuration. This was done at `compiler-core/src/config.rs`. Right now, it includes the `[glistix.preview]` section, which contains:
    1. `[glistix.preview.hex-patch]`: allows specifying dependency metadata to override when publishing to Hex. For example, specify `package = ">= 0.34.0 and < 2.0.0"` under this section to tell Hex you depend on that package with that version (**that will be your effective dependency** for packages which depend on yours via Hex), whereas in `[dependencies]` you depend on the package to be available locally (only effective when developing the package locally or running tests etc.). **This is a temporary workaround while we don't have proper dependency patching.** It is necessary because the stdlib needs to be patched to include Nix support, which is done through local dependencies on Git submodules.
        - Relevant implementation is at `compiler-cli/src/publish.rs`.

    2. `local-overrides = [ ... ]` (directly under `[glistix.preview]`): allows listing local dependencies which **should take priority over transitive local dependencies with the same name**. By default, Gleam (and thus Glistix) errors when you have two local dependencies with the same name pointing to different paths; this setting overrides that such that **the root local dependency always wins**. If the root package doesn't have this dependency or doesn't set `local-overrides`, the error can still occur. **This is also a workaround while Gleam doesn't have proper dependency patching.**

- We have added **Nix syntax highlighting support** to **packages' generated documentation** (through `glistix docs`). The relevant `highlight.js`-compatible file is at `compiler-core/templates/docs-js/highlightjs-nix.min.js`.

- It is worth noting that we have **reutilized most of the Gleam compiler's GitHub Actions workflows** for our own usage in CI and other tasks. However, **noteworthy changes were made** to better adapt them to Glistix's needs.

- We have **modified language tests** (in `test/language`) to **support the Nix target**. These tests run on CI.
