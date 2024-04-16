# Sharing and using your package

Great, you've made a neat little project which you now want to use, maybe for some Nix derivation you're configuring in some other repository, or for your NixOS configuration, for example. Your `main` function under `src/packagename.gleam` is ready and returns some functions and values in an attribute set which you plan to use in those projects.

How will your other projects access this?

Luckily for us, `glistix new` creates a `flake.nix` for us which automatically exports the following outputs:

1. `packages.<system>.default`: A derivation which **builds your Glistix project from scratch** (this uses the Glistix compiler, which might have to be built from source as well). The derivation's `out` output contains a `nix` folder with everything that ends up in the build directory under the `nix` target, including `packagename.nix` (with your `main` lambda exported, for example) as well as other Nix files (compiled from your source Gleam files).

    - While using this package is ideal, it's worth noting the overhead of having to compile Glistix _and_ your project from scratch, which is why we chose to present the alternative below as well.

2. `lib.runGlistixPackage` (TBD): A lambda with named arguments (or `{}` to just use the defaults) which is prepared for a setup in which you **commit your build output to the repository** for ease of use (so you don't have to specify the system or build Glistix to use your generated Nix code). By default, it checks the `output/nix` folder; if it exists, calls `import` on `packagename.nix` by default and returns the output of running the exported main function. If it doesn't exist, it will fall back to building your project using the given `pkgs`, if available; otherwise, the given `system`, if available; or `builtins.currentSystem`, if that's available as well. If all options fail, importing your package will fail.

    - This is better for users of your package, as they don't have to compile using Glistix (or Glistix itself) before using your project, but might incur a bit of maintenance overhead, since you'll have to manually run `mv build/dev/nix -T output/nix` (that is, move the results of `glistix build` to `output/nix`) and commit the changes for each completed build you'd like your Nix library users to use.

This page is under construction.
