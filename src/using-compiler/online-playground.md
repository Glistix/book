# Online playground

Glistix has an online playground available at [https://glistix.github.io/playground](https://glistix.github.io/playground). Give it a try!

**Without installing anything,** the playground can be used to:

1. **Quickly convert some Gleam code to Nix code using Glistix.** Just type the Gleam code on the left and the Nix code appears on the "Compiled Nix" tab to the right.
2. **Quickly share some Gleam code compiled to Nix to the web.** After typing the Gleam code to the left, just press the "Share code" button to the right, and you will get a permanent link to the playground with your code.

Please note that it currently has the following **limitations:**

1. **The only package available is [the Glistix port of Gleam's `stdlib`](https://github.com/glistix/stdlib)**. As such, you cannot import other packages online just yet.
2. **The "Output" tab uses the JavaScript target, as it's the only target that runs in the browser.** As such, when you're using Nix-exclusive features, that tab will display some errors you can ignore. What matters is the "Compiled Nix" tab.

**Found an issue with the playground?** Feel free to [open an issue at the playground's repository.](https://github.com/glistix/playground)
