# Import a package from Nix

Great, you've made an awesome Glistix project which you now **want to use in Nix,** maybe for some Nix derivation you're configuring in some other repository, or for your NixOS configuration, for example. Your `main` function under `src/packagename.gleam` is ready and returns some functions and values in an attribute set which you plan to use in those projects (again an example).

Now, **how will your other Nix projects access your Gleam `main` function?** (Or any other function!)

Luckily for us, `glistix new` **automatically creates a `flake.nix` file** for our new project, as well as `default.nix`. This makes it **easy to import your Glistix project as part of another Nix project:** everything you need **is exported by these two Nix files** (the former with [`builtins.getFlake`](https://nixos.org/manual/nix/stable/language/builtins.html#builtins-getFlake) or as a flake input of another flake, the latter with [`import`](https://nixos.org/manual/nix/stable/language/builtins.html#builtins-import)).

This is because both files (with `default.nix` just mirroring `flake.nix`) export `lib.loadGlistixPackage`. This is a **Nix function** with the sole purpose of **importing your Gleam code transpiled to Nix**. You simply call it with `lib.loadGlistixPackage { }` and, by default, **it will import your package's main module** (with the same name as your package, which usually has the `main` function). You can **pick the imported module** with `lib.loadGlistixPackage { module = "my/module"; }`, for example.

It does that by **invoking the Glistix compiler to build your project from scratch**, and then importing the resulting build folder (moved to somewhere at `/nix/store`). However, since that process depends on the Glistix compiler, **that might trigger a full compilation of Glistix itself, which is slow.** Therefore, **you might want to cache the built Nix files in your repository** to speed up the process - more at the next section.

As an example, let's assume your Glistix project is stored in a **subfolder of your main Nix project.** You can then import it like this:

```nix
let
  yourProject = import ./project/folder;
  yourPackage = yourProject.lib.loadGlistixPackage { };
  mainResult = yourPackage.main { };  # call main()
in { inherit mainResult; } # use for whatever you want!
```

With Flakes, you'd add your project as an input. For example:

```nix
{
  inputs = {
    yourProject.url = "github:your/repo";
    # ...
  };

  outputs = inputs@{ yourProject, somethingElse, ... }:
    let
      # You can import functions from other modules, not just main.
      # 'system' is needed here so the correct Glistix derivation
      # is used to build your package.
      yourModule = system: yourProject.lib.loadGlistixPackage {
        inherit system; module = "mod/name";
      };
      funcResult = system: (yourModule system).add 50 12;
    in
    {
      # Use it as you wish!
      # For example, within your NixOS configuration,
      # or as a parameter to a builder, or anything else, really.
      outputName.x86_64-linux.default = somethingElse (funcResult "x86_64-linux");
      outputName.x86_64-darwin.default = somethingElse (funcResult "x86_64-darwin");
    }
}
```

And that's it! You can now use your Gleam project within other Nix projects.

<div class="warning">

When invoking a Gleam function from Nix, **make sure to use the correct representation of Gleam's types,** as well as **follow the conventions for calling Gleam functions.** Read the ["Nix Target" chapter](../nix/target) for more information.

</div>

### Caching your built Nix files

It is worth noting, however, that, by default, **your project is built from scratch each time it is loaded,** which **requires Glistix.** Since machines might not have Glistix installed, **the first load might require building Glistix from source** (as Glistix is not currently available on `nixpkgs`). That's also why the `system` parameter is needed when evaluating in pure mode (the default for flakes), as `builtins.currentSystem` (the default) fails.

To avoid that problem and have your Glistix project not depend on Glistix for Nix consumers, you can **cache the built Nix files.** To do so, **trigger a Glistix build locally** - for example, `glistix build -t nix` - and then **copy the build results to an `output/` directory**, like so:

```sh
# Create destination
mkdir -p output/dev

# Copy just the resulting Nix files
# (IMPORTANT: -L flag used to deep copy symlinked folders in 'build')
cp -rL build/dev/nix -t output/dev

# Check it in so the flake can access and import from the folder
git add output
```

The `-L` flag is needed as the Glistix compiler (and the upstream Gleam compiler, for that matter) generates symlinks at the build directory to each package's `priv/` folder (if it has one). This option will ensure the symlinks are properly resolved into actual files and folders (they are duplicated from the build directory).

Once that's in place, **future calls to `lib.loadGlistixPackage { }` will automatically pick up your `output/` folder, skipping the build process entirely** for downstream Nix users, as long as it's properly set up (`output/dev/nix` is present and added to Git). **Otherwise** (if not properly set up), **it will fallback to building from scratch.** (You can force it to not do that by passing `{ derivation = null; }` - the `flake.nix` has a user-friendly option at the top to enable that. Then it will just error instead.)

The major downside, of course, is **having to keep the `output/` folder up-to-date** whenever you make relevant changes to your Gleam code. The tradeoffs should be considered.

### Description of the generated flake outputs



1. `packages.<system>.default`: A derivation which **builds your Glistix project from scratch** (this uses the Glistix compiler, which might have to be built from source as well). The derivation's `out` output contains a `dev/nix` folder from Glistix's build output, including `packagename.nix` (with your `main` lambda exported, for example) as well as other Nix files (compiled from your source Gleam files).

    - This uses Glistix's `builders.<system>.buildGlistixPackage` output under the hood.

    - While using this package is theoretically ideal, in practice, the overhead of having to **compile Glistix _and_ your project from scratch** is noticeable, which is why we made an alternative: caching the build outputs and using `lib.loadGlistixPackage`.

2. `lib.loadGlistixPackage`: A lambda with named arguments (or `{ }` to just use the defaults) which is prepared for a setup in which you **commit your build output to the repository** for ease of use (so you don't have to specify the system or build Glistix to use your generated Nix code). By default, it checks the `output/nix` folder; if it exists, calls `import` on `packagename.nix` by default and returns the output of running the exported main function. If it doesn't exist, it will fall back to building your project using the given `pkgs`, if available; otherwise, the given `system`, if available; or `builtins.currentSystem`, if that's available as well. If all options fail, importing your package will fail.

    - This is better for users of your package, as they don't have to compile using Glistix (or Glistix itself) before using your project, but might incur a bit of maintenance overhead, since you'll have to manually run `mv build/dev/nix -T output/nix` (that is, move the results of `glistix build` to `output/nix`) and commit the changes for each completed build you'd like your Nix library users to use.

This page is under construction.
