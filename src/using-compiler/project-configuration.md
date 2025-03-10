# Project Configuration

You can configure your project through a `gleam.toml` file. Its settings are mostly the same as Gleam's usual settings, available at [https://gleam.run/writing-gleam/gleam-toml/](https://gleam.run/writing-gleam/gleam-toml). However, **Glistix additionally defines a few extra settings:**

1. There is a new `[glistix.preview]` section for temporary settings while Glistix is in beta. It is expected that those settings will change in the future.

2. It contains a new `[glistix.preview.patch]` section, which is similar to `[dependencies]`, however it is used to **override transitive dependencies with other packages.** This can be used to replace dependencies with Nix-compatible forks. Note that this setting is only read for top-level projects, and ignored for libraries; end users must patch all necessary transitive dependencies by themselves. (More information at ["Overriding incompatible packages"](../recipes/overriding-packages.md).)

    For example:

    ```toml
    [dependencies]
    # When published to Hex, depend on the normal stdlib instead.
    # Downstream users will have to patch manually.
    gleam_stdlib = ">= 0.34.0 and < 2.0.0"

    [glistix.preview.patch]
    # When developing locally, replace 'gleam_stdlib' with 'glistix_stdlib' from Hex
    # on all transitive dependencies.
    gleam_stdlib = { name = "glistix_stdlib", version ">= 0.34.0 and < 2.0.0" }
    # Can also replace with local packages.
    gleam_json = { name = "glistix_json", path = "./external/json" }
    ```
