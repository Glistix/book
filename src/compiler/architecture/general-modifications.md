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
    1. `[glistix.preview.patch]`: allows overriding transitive dependencies with Nix-compatible forks. For example, specify `gleam_stdlib = { name = "glistix_stdlib", version = ">= 0.34.0 and < 2.0.0" }` to replace the incompatible package `gleam_stdlib` with the Nix-compatible `glistix_stdlib` published on Hex.
        - Note that this section is ignored when publishing to Hex, and only dependencies on `[dependencies]` will be read. End users must patch any transitive dependencies by themselves.
        - More details in [the configuration book page](../../using-compiler/project-configuration.md).
        - This was implemented in [#44](https://github.com/Glistix/glistix/pull/44).
        - Relevant implementation is spread across `compiler-core/src/config.rs` (contains the main patching logic, as well as detection of patched packages in the manifest to be unlocked after patch changes), `compiler-core/src/dependency.rs` (patches transitive dependencies from Hex), `compiler-core/src/manifest.rs` (manifest now stores patches used when building it), `compiler-core/build/project_compiler.rs` (patches module names at build time), `compiler-cli/src/config.rs` (invokes patching of the root config and dependency configs on `read`), `compiler-cli/src/dependencies.rs` (reads the root config, patches local dependencies and invokes hex dependency patching, as well as detects when patches have changed and the manifest has to be updated, and stores patches in the manifest), `compiler-core/src/language_server/router.rs` (patches config read by the language server) and `compiler-cli/src/add.rs` (now has to resolve package versions twice when the added package was already patched: once without the patch to pick a version for `[dependencies]`, and once again with the patch to fix the `manifest.toml`).

    2. Two other settings, `local-override` and `[glistix.preview.hex-patch]`, were previously used to select transitive local dependencies to be overridden by the root package's local dependencies in case of conflict and to override dependencies listed when publishing to Hex, respectively. They are now deprecated in favor of `[glistix.preview.patch]`, which can be used for both purposes (you can patch transitive local dependencies to ensure they don't conflict with the root dependencies, and `[dependencies]` are the dependencies effectively published to Hex and can be patched separately).

- We have **added some more useful error hints**, as well as **Glistix-exclusive errors,** at `compiler-core/src/error.rs`.

- We have added **Nix syntax highlighting support** to **packages' generated documentation** (through `glistix docs`). The relevant `highlight.js`-compatible file is at `compiler-core/templates/docs-js/highlightjs-nix.min.js`.

- It is worth noting that we have **reutilized most of the Gleam compiler's GitHub Actions workflows** for our own usage in CI and other tasks. However, **noteworthy changes were made** to better adapt them to Glistix's needs.

- We have **modified language tests** (in `test/language`) to **support the Nix target**. These tests run on CI.
