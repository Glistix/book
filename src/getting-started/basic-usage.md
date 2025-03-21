# Basic Usage

Before you start, note that you can **give Glistix a try in your browser** without installing anything! You can try it out in the playground at [https://glistix.github.io/playground](https://glistix.github.io/playground). Read the [book page about the playground](../using-compiler/online-playground.md) for more information.

Once you're ready, make sure to [install the Glistix compiler to your computer](./installation.md). Afterwards, here's how you can start working on a new Glistix project straight away:

1. Use the `glistix new NAME` command to create a new Glistix project.
    - This command will set (almost) everything up for you, including initialize a Git repository, initialize the project structure (`gleam.toml`, `src/`, `test/` etc.), prepare essential `*.nix` files, and even clone Glistix's standard library to `external/stdlib` as a Git submodule (this is a workaround which is currently needed while we don't have Git dependencies!).

    - We also **generate a default GitHub Actions CI workflow file** which tries to build your project through `flake.nix`. You can add `--skip-github` to `glistix new` to opt out of the creation of this file (or just delete it).

2. You can edit `src` to customize the Gleam code, as well as edit `gleam.toml` to your liking.

    - You can use `glistix add name` to add a dependency from Hex. For instance, you may want to **use the [`glistix_nix` package](https://github.com/glistix/nix) to easily access certain Nix built-in types** from Gleam, which can be done with `glistix add glistix_nix`.

        - If your desired dependency doesn't support Nix, **you will have to use a fork patched for Nix support.** Check ["Overriding incompatible packages"](../recipes/overriding-packages.md) for more information.

    - Note that Git dependencies are not yet supported by Glistix. If you need those, you'll have to clone them as submodules and use as local dependencies. See [Limitations](../about/limitations.md) for more information.

3. Run `glistix build` at least once, not only to make sure everything is working, but also to **generate the `manifest.toml`** (which should be checked into your repository).

4. Afterwards, to complete the Nix side of your setup, ensure you have [Nix with flakes support](https://wiki.nixos.org/wiki/Flakes) available (the `nix` command), as well as run `git add .` so all relevant files are checked in, and then run the command below to generate your `flake.lock`.

    ```sh
    nix flake update
    ```

Nice! Your project is now **ready to be used** by both Nix users (which will use your Gleam code compiled to Nix) and also other Glistix users.

To **import a Gleam module in your project from within Nix,** the `default.nix` and `flake.nix` files in your new project export a `lib.loadGlistixPackage { module = "module/name"; }` function, which, when used, will give you an attribute set with all names exported by that module, so you can use its record constructors, constants and functions from within Nix. See ["Import a Gleam package in Nix"](../recipes/import-in-nix.md) for more information.
