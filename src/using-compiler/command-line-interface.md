# Command-Line Interface

Here are some of the most common compiler commands.

1. `glistix new`: An essential command, creates a new Glistix project for you with batteries included, containing:
    - Several basic directories and files of the project structure expected by the compiler;
        - This includes an initial `gleam.toml` tuned for Glistix-specific defaults;
    - An initial `flake.nix` (see ["Sharing and using your package"](./sharing-package.md));
    - GitHub Actions workflows for CI, as well as releasing a new version of your package (triggered by creating a `v{version}` tag);
    - An initial Git repository with a `.gitignore` file.

2. `glistix build [--target nix]`: Builds your project to `build/dev/<target>`, by default the target specified in your `gleam.toml` unless you specify `--target <target>`.
    - Note that, if no target is specified in your `gleam.toml`, the compiler **will default to the `erlang` target** to be compatible with existing Gleam projects. As such, make sure to specify `target = "nix"` in your `gleam.toml`.

3. `glistix run [--target nix]`: Runs your project's main function in the target specified either by `--target` or by `gleam.toml` (or `erlang` by default, for the same reason as before).
    - For the Nix target, this will call `nix-instantiate` to evaluate your `packagename.gleam`'s `main` function.

4. `glistix test [--target nix]`: Similar to `glistix run`, but runs the main function compiled from `test/packagename_test.gleam`.

5. `glistix format`: Formats your Gleam code according to Gleam's standards.

6. `glistix clean`: Deletes your `build/` directory. Useful to get rid of stale package versions.

7.  `glistix add name`: Adds a [Hex](https://hex.pm) dependency to your project.

8. `glistix docs build`: Builds documentation for your package to `build/docs`.

9. `glistix publish`: Publishes your package to Hex.
