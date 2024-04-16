# Project Structure

When you run `glistix new`, the compiler will generate a project which should conform to the following structure (largely based on Gleam's structure, which by itself is based on [Erlang's project structure](https://www.erlang.org/doc/design_principles/applications.html#id80846)):

1. `gleam.toml` contains all essential information regarding your project that the compiler should be aware of, including metadata (such as package name), your preferred target (defaults to `nix`, can also be `javascript` or `erlang` for compatibility with other Gleam projects), and also specifying your dependencies.
2. `src/` contains the source code of your package. This can contain both `.gleam` files and also FFI files (`.nix` for the Nix target). A `packagename.gleam` file with a `main` public function with zero arguments is expected if your package is not a Gleam library (but rather made to be used within Nix). If present, you can check its output with `gleam run`.
3. `test/` optionally contains a `packagename_test.gleam` file containing a single `main` function with zero arguments which
is called when running `gleam test`. Use this with [`glistix_gleeunit`](https://github.com/glistix/gleeunit) or some other test runner.
4. `priv/` is an optional folder for assets and other general files needed by your project and is not present by default.
5. `include/` is an optional folder that other apps using your project should be aware of and is not present by default.
