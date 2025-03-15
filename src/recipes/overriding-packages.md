# Overriding incompatible packages

Many **existing Gleam packages** available over Hex **will not work on Nix** by default, as **they rely on FFI with Gleam's typical targets** (Erlang and JavaScript). Most importantly, **this includes Gleam's standard library** ([`gleam_stdlib`](https://github.com/gleam-lang/stdlib)).

The workaround is to **create and/or use Nix-compatible forks** of those packages. For instance, [**Glistix maintains its own fork of Gleam's standard library,** called `glistix_stdlib`](https://github.com/glistix/stdlib).

If those forks are **published on Hex** (which is the case of Glistix's forks, for example), then using them is **easy** - you just need to add the following line(s) under `[glistix.preview.patch]` to your configuration, and then **all of your dependencies and transitive dependencies will be using the fork** instead of the original package, providing Nix target support:

```toml
# Example: patch gleam_stdlib with glistix_stdlib
[glistix.preview.patch]
gleam_stdlib = { name = "glistix_stdlib", version = ">= 0.34.0, < 2.0.0" }
```

Regarding `gleam_stdlib` specifically, don't worry - the `glistix new` command **automatically suggests patching `gleam_stdlib`** (the lines shown above are already written to the config by default). However, you will still have to add more patches by yourself for any other package patches you - or your dependencies - might need (e.g. [`glistix_json`](https://github.com/glistix/json) as a replacement for `gleam_json`).

You will need to add more such lines under `[glistix.preview.patch]` in the configuration of your **top-level Glistix project** until all Nix-incompatible packages have been replaced with compatible forks (you may have to create your own if no existing forks are available).

The compiler tries to add hints to compilation errors when it guesses that a dependency probably needs a patch, but in general, a dependency which fails to compile is a strong sign of incompatibility with Nix.

<div class="warning">

Note that a **dependency failing to compile** could also indicate that **you applied a patch which a dependency is incompatible with!** For example, if a dependency needs `gleam_stdlib` v0.37 and you applied a patch for `glistix_stdlib` v0.38, you might receive an error when compiling that dependency.

</div>

Please note that **patches are ignored when publishing a package to Hex** - they are only applied on **top-level projects** (those which you can `glistix run`, so, not libraries). End users are responsible for patching transitive dependencies by themselves. Therefore, it could be interesting to **warn about patches in your package docs/README** in case there are confused users of your library (assuming you are creating a library). However, **patches are still useful for packages** in order to allow the package authors to execute tests with `glistix test` (test modules count as part of a top-level project).

However, if the desired fork is not published on Hex, the above process is **not enough**: you will need to use patches to local dependencies corresponding to **Git submodules** cloned to an `external/` folder, as a workaround while Glistix does not yet support Git dependencies (they were implemented upstream on Gleam 1.9.0, which hasn't made its way to Glistix's codebase yet). This is tracked by Glistix issue [#47](https://github.com/Glistix/glistix/issues/47).

In the meantime, the full instructions remain below.

## Steps to override a package using Git submodules

Ideally, overriding a package with a Nix-compatible fork is as simple as **patching the package to a fork published on Hex.** If the fork you want to use is on Hex, then **please use the instructions above instead** and **skip this section entirely.**

However, if the fork is only available on a Git repository (such as GitHub), a separate workaround is needed, through **Git submodules.** This is only while Git dependencies are not yet available on Glistix.

As an example, the [official `gleam_json` package](https://github.com/gleam-lang/json) does not support the Nix target by default, while several Gleam packages depend on it, creating a problem if we want to use those packages. Luckily, the Glistix project maintains a Nix-compatible fork of this package, `glistix_json`, at [https://github.com/glistix/json](https://github.com/glistix/json).

Since [`glistix_json` is available on Hex](https://hex.pm/packages/glistix_json), we can use the procedure described in the previous section to apply a patch from `gleam_json` to `glistix_json`, ensuring we can use packages which depend on `gleam_json` (and also so we can depend on it ourselves).

However, **for demonstration purposes only,** here's how we could use it as a Git submodule (you'd apply the same instructions for forks not available on Hex):

1. Run the command below to add the repository as a Git submodule of your project. We add submodules to the `external/` folder as a convention:

    ```sh
    git submodule add --name json -- https://github.com/glistix/json external/json
    ```

2. Add `glistix_json` to your `gleam.toml` as a patch of `gleam_json` pointing to the `glistix_json` submodule you cloned locally:

    ```toml
    [glistix.preview.patch]
    gleam_json = { name = "glistix_json", path = "./external/json" }
    ```

    Please note that this patch, much like a Hex patch, is ignored if you publish your package to Hex. This section is read for top-level packages used by `glistix run`, `glistix build` and `glistix test`, but not for any dependencies or libraries, as previously mentioned.

3. **Optional:** If you also want to use `gleam_json` in your own project, feel free to add it as a regular dependency of your project at `gleam.toml`:

    ```toml
    [dependencies]
    gleam_json = ">= 2.0.0, < 3.0.0"
    ```

4. Finally, make sure to **update your `flake.nix` file** so that it will correctly clone the submodule when building your package through Nix. You can do so by **adding the fork's repository as a Flake input,** and then **passing its downloaded source to the `submodules` list** (which is later used as an argument to `buildGlistixPackage`), as below:

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

1. `glistix build`, to update the `manifest.toml` and ensure your package builds;

2. `nix build`, to ensure your package can still be built from the flake.
