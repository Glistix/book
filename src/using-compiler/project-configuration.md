# Project Configuration

You can configure your project through a `gleam.toml` file. Its settings are mostly the same as Gleam's usual settings, available at [https://gleam.run/writing-gleam/gleam-toml/](https://gleam.run/writing-gleam/gleam-toml). However, **Glistix additionally defines a few extra settings:**

1. There is a new `[glistix.preview]` section for temporary settings while Glistix is in beta. It is expected that those settings will change in the future.

2. Within it, you can define `local-overrides = ["list", "of", "packages"]`. For example:

    ```toml
    [glistix.preview]
    local-overrides = ["gleam_stdlib"]
    ```

    This is used in case of **conflicts between local dependencies**. When the root package (the project) being compiled specifies some packages in that list, and the project itself depends on those packages, then the project's local dependencies are prioritized over the conflicting transitive local dependencies. (Motivation at ["Overriding incompatible packages"](../recipes/overriding-packages.md).)

3. There is also a new `[glistix.preview.hex-patch]` section, which is similar to `[dependencies]`, however it **only applies when publishing your package to Hex.** This is a workaround so you can have local dependencies to Nix-compatible forks of packages, but still be able to publish a package to Hex. (More at ["Overriding incompatible packages"](../recipes/overriding-packages.md).)

    For example:

    ```toml
    [dependencies]
    # When developing locally, use stdlib patch at that path.
    gleam_stdlib = { path = "./external/stdlib" }

    [glistix.preview.hex-patch]
    # When published to Hex, depend on the normal stdlib instead.
    # Downstream users will have to patch manually.
    gleam_stdlib = ">= 0.34.0 and < 2.0.0"
    ```
