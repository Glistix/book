# Overriding incompatible packages

Many **existing Gleam packages** available over Hex **will not work on Nix** by default, as **they rely on FFI with Gleam's typical targets** (Erlang and JavaScript). Most importantly, **this includes Gleam's standard library** ([`gleam_stdlib`](https://github.com/gleam-lang/stdlib)).

The workaround is to **create Nix-compatible forks** of those packages. For instance, [**Glistix maintains its own fork of the standard library**](https://github.com/glistix/stdlib). To use them, however, we have the following **challenges:**

1. Currently, **Gleam does not have a proper requirement overriding system** (tracked at [upstream issue #2899](https://github.com/gleam-lang/gleam/issues/2899)). As such, we have to use those forks as **local or Git dependencies.** That way, **they override transitive Hex dependencies with the same name,** thus ensuring the fork is used instead of the Hex version, even if the package appears as a transitive dependency (i.e. a dependency of a dependency, at any level).

2. However, **Gleam does not currently support Git dependencies** (tracked at [upstream issue #1338](https://github.com/gleam-lang/gleam/issues/1338)). As such, **we depend on local paths pointing to cloned Git submodules.**

3. Finally, **each fork needs to have the same name as the package it's patching** (due to a compiler requirement, while there is no built-in patching yet) and, as such, **cannot be published to Hex** (as it can't replace the patched package itself, of course).

It is expected that a proper solution to these problems will be available in the future. For now, the best we have is **cloning forks as Git submodules and using them as local dependencies.**

Don't worry, though - `glistix new` **automatically patches `gleam_stdlib` for you** by setting up `gleam.toml` and cloning it as a submodule to `external/stdlib`. However, you will have to do that by yourself for any other package patches you might need (e.g. [`glistix/json`](https://github.com/glistix/json)).

## Steps to override a package

The [`gleam_json` package](https://github.com/gleam-lang/json), for example, does not support the Nix target by default, while several Gleam packages depend on it, creating a problem if we want to use them. Luckily, the Glistix project maintains a Nix-compatible fork of this package at [https://github.com/glistix/json](https://github.com/glistix/json). Here's how we can use it, ensuring we can use packages which depend on `gleam_json` (and also so we can depend on it ourselves):

1. Run the command below to add the repository as a Git submodule of your project. We add submodules to the `external/` folder as a convention:

    ```sh
    git submodule add --name json -- https://github.com/glistix/json external/json
    ```

2. Add `gleam_json` as a local dependency to that submodule at your project's `gleam.toml`:

    ```toml
    [dependencies]
    gleam_json = { path = "./external/json" }
    ```

3. To ensure your local override of `gleam_json` takes precedence over transitive local dependencies (see explanation at ["Dealing with local package conflicts when patching"](#dealing-with-local-package-conflicts-when-patching)), you can add `gleam_json` to `local-overrides` at `[glistix.preview]`, as below:

    ```toml
    [glistix.preview]
    # note that 'gleam_stdlib' is there by default
    local-overrides = ["gleam_stdlib", "gleam_json"]
    ```

4. Additionally, **if you intend on publishing your package to Hex,** which doesn't support local dependencies, you will have to use the (temporary) workaround below to tell Hex you depend on `gleam_json` from Hex instead, as explained [in the next section](#publishing-to-hex-with-patches).

    ```toml
    [glistix.preview.hex-patch]
    gleam_json = ">= 1.0.0 and < 2.0.0" # for example
    ```

5. Finally, make sure to **update your `flake.nix` file** so that it will correctly clone the submodule when building your package through Nix. You can do so by **adding the fork's repository as a Flake input,** and then **passing its downloaded source to the `submodules` list** (which is later used as an argument to `buildGlistixPackage`), as below:

    ```nix
    # At your flake.nix
    {
      inputs = {
        # ...

        json = {  # <-- add this input
          url = "github:glistix/json";
          flake = false; # <-- get the source, not the flake
        };
      };

      outputs =
        inputs@{
          # ...
          json, # <-- add this argument
          # ...
        }:
        let
          # ... initial definitions here ...
          submodules = [
            {
              src = stdlib;
              dest = "external/stdlib";
            }
            {
              src = json;  # <-- add this here
              dest = "external/json";
            }
          ];
          # ...
        in
        {
          # ...
        };
    }
    ```

5. Run `nix flake lock` to update your `flake.lock` to include the new submodule input.

Finally, make sure everything is working by running:

1. `glistix build`, to **update the `manifest.toml`** and **ensure your package builds**;

2. `nix build`, to **ensure your package can still be built from the flake.**

## Publishing to Hex with patches

Normally, **patching packages wouldn't let you publish your packages to Hex**, since you cannot depend on local packages as a Hex package (you must depend on other Hex packages). Therefore, as a temporary workaround, **Glistix added a `[glistix.preview.hex-patch]` setting to `gleam.toml`**, where you can **override your package's dependencies once published to Hex.** This allows you to **replace a local dependency with a Hex dependency, but only when publishing.** For example:

```toml
# ...
[dependencies]
# While developing, depend on your local patch of the stdlib...
gleam_stdlib = { path = "./external/stdlib" }

[glistix.preview.hex-patch]
# However, once published to Hex, your package will depend
# on the regular stdlib instead, for compatibility with
# other packages.
gleam_stdlib = ">= 0.34.0 and < 2.0.0"
```

As such, **users of your package will be responsible for patching** by themselves, but we expect patching to only be truly necessary for core packages (e.g. `gleam_stdlib`), which most Glistix users will have to patch anyway (until we get proper requirement overriding).

## Dealing with local package conflicts when patching

The procedure above works as **local dependencies always override transitive Hex dependencies.** However, **they do not override other local dependencies with the same name** by default.

For example, if you clone both `glistix/stdlib` and `glistix/json` repositories as submodules to `external/stdlib` and `external/json` respectively, and depend on them on `gleam.toml` via `gleam_stdlib = { path = "./external/stdlib" }` and `gleam_json = { path = "./external/json" }` respectively, **you will get an error** because `gleam_json` depends on `gleam_stdlib` at `external/json/external/stdlib`, while you depend on `gleam_stdlib` at `external/stdlib`. In other words, **there are conflicting local dependencies.**

To solve this, while a proper requirement overriding system isn't in place, **you can, as a temporary workaround, add the setting below** to the root package's `gleam.toml` (i.e. your project's `gleam.toml`) so that its version of `gleam_stdlib` is prioritized by the compiler:

```toml
[glistix.preview]
# Ensure the root package's local dependency
# on 'gleam_stdlib' is prioritized.
local-overrides = ["gleam_stdlib"]
```

That way, Glistix will know how to solve the conflict (use `gleam_stdlib` from `external/stdlib`, not from `external/json/external/stdlib`). Note that **this setting is ignored for non-root packages** (dependencies cannot apply `local-overrides`).
