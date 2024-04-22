# CLI Modifications

We have changed the following parts of the CLI (crate `compiler-cli`):

1. `new.rs` (`glistix new`):
    - Added Nix-relevant file templates (`default.nix`, `shell.nix` and `flake.nix`);
    - Clone [`glistix/stdlib`](https://github.com/glistix/stdlib) to `externals/stdlib` as a Git submodule by default;
    - Changed default `gleam.toml` to include Glistix-specific options.
2. `run.rs` (`glistix run`, `glistix test`):
    - Use `nix-instantiate` when calling `glistix run` or `glistix test` on the Nix target.
3. `publish.rs` (`glistix publish`):
    - Implement `[glistix.preview.hex-patch]` by replacing dependencies with what's specified in `hex-patch` right before publishing.
4. `dependency.rs` (resolving versions for the `manifest.toml`):
    - Implement `local-overrides` from `[glistix.preview]` by replacing provided (local/Git) dependencies with what the root package specified for them, if overridden by the root.
5. `fs.rs`: Added Git operations used by `new.rs`.
