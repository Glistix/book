# Installation

Glistix officially supports **Linux, MacOS and Windows.** (Note, however, that Nix doesn't support Windows yet, so you won't be able to test your projects, but `glistix build` should work at least.)

In addition, Glistix supports **running in the browser (using WebAssembly).** You can try it out [in the playground](../using-compiler/online-playground.md) without installation: [https://glistix.github.io/playground](https://glistix.github.io/playground)

You can install Glistix to your computer in one of the following ways.

1. **From GitHub Releases:** Regardless of your OS or distribution, you can install Glistix by downloading the latest precompiled binary for your platform at [https://github.com/glistix/glistix/releases](https://github.com/glistix/glistix/releases).

2. **With Nix flakes:** Invoke the command below in the command line to download, compile and run a specific release of Glistix - here the latest at the time of writing (v0.8.0).

    ```sh
    nix run 'github:Glistix/glistix/v0.8.0' -- --help
    ```

    To install permanently, you can either add `github:Glistix/glistix/v0.8.0` as an input to your system/Home Manager configuration, or use `nix profile`:

    ```sh
    nix profile install 'github:Glistix/glistix/v0.8.0'
    ```

3. **With Cargo:** You can use Cargo to compile and install Glistix's latest release (v0.8.0 at the time of writing):

    ```sh
    cargo install --git https://github.com/glistix/glistix --tag v0.8.0 --locked
    ```

Currently, Glistix cannot be installed through nixpkgs.
