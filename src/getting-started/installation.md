# Installation

You can install Glistix in one of the following ways.

1. **With Nix flakes:** Invoke the command below in the command line to download, compile and run a specific release of Glistix - here the latest at the time of writing (v0.1.0).

    ```sh
    nix run github:Glistix/glistix/v0.1.0 -- --help
    ```

2. **With Cargo:** You can use Cargo to install Glistix's latest release (v0.1.0 at the time of writing):

    ```sh
    cargo install --git https://github.com/glistix/glistix --tag v0.1.0 --locked
    ```

Currently, Glistix cannot be installed through nixpkgs.
