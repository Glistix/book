# Contributing

You may contribute in many ways. The compiler's source code, **in Rust**, is publicly available at its [GitHub](https://github.com/glistix/glistix) and [Codeberg](https://codeberg.org/glistix/glistix) mirrors. You may contribute to it by first **browsing (or submitting) issues** and then **submitting pull requests** at either of its mirrors.

However, **you don't need to know Rust to contribute.** If you know some Nix, **please give us a hand with porting Gleam packages to Nix**. For example, [the stdlib port is currently incomplete](https://github.com/glistix/stdlib).

Please **discuss first** before working on a major contribution. The ideal channels for this are:
1. [GitHub issues](https://github.com/Glistix/glistix). Feel free to open an issue proposing a feature and we can discuss it there!
2. [Our Zulip chat](https://glistix.zulipchat.com). To join, you may use the invite link at the [introduction page](../).
    - Please [open an issue in the book repository](https://github.com/Glistix/book) if the link doesn't work for you.

Additionally, make sure to **read the rest of the book** for useful information on how the compiler works. [The "Glistix Architecture" chapter](../compiler/architecture) goes into great insight in this regard, and is a valuable resource.

Please note that we generally try to follow an **upstream-first policy.** This means that any feature ideas which are not related to Nix should generally be brought to the the upstream Gleam repository first; if accepted there, the feature should be implemented (/contributed) upstream. Otherwise, we can consider implementing it on Glistix if it would significantly improve the experience of using Glistix, especially regarding usage with Nix, but not before proper discussions. The idea is to not only keep up with the Gleam ecosystem at large, but also to ensure most improvements to the Glistix compiler also benefit users of the upstream Gleam compiler.
