# CLI Modifications

We have changed the following parts of the CLI (crate `compiler-cli`):

1. `new.rs` (`glistix new`):
    - Added Nix-relevant file templates (`default.nix`, `shell.nix` and `flake.nix`);
    - Changed default `gleam.toml` to include Glistix-specific options.
      - It now patches `gleam_stdlib` to [`glistix_stdlib`](https://github.com/glistix/stdlib) by default on `[glistix.preview.patch]`.
      - See more information at the [configuration book page](../../using-compiler/project-configuration.md).
2. `run.rs` (`glistix run`, `glistix test`):
    - Use `nix-instantiate` when calling `glistix run` or `glistix test` on the Nix target.
3. `publish.rs` (`glistix publish`):
    - Uses `read_unpatched` to read configurations, thus using `[dependencies]` as the published Hex dependencies, ignoring patches from `[glistix.preview.patch]`.
4. `dependency.rs` (resolving versions for the `manifest.toml`):
    - Implement features related to `[glistix.preview.patch]`, including detecting if patches changed before updating the manifest, ensuring local and Git dependencies specified through patches are provided and available, passing patch information to Hex dependency resolving, and storing patches in the manifest.
5. `add.rs`:
    - Now has an additional dependency resolving step when the added package was affected by a patch (first, a version is resolved ignoring patches, generating the wrong manifest, and then we fix the manifest, keeping the initially resolved version in `[dependencies]`).
6. `config.rs`: Invokes patching when reading configuration files as necessary.

`glistix build` (`build.rs`) wasn't directly modified, but it now supports `--target nix` as well.
