# Goals & Roadmap

## Glistix's Goals

Glistix has the following **primary goals**:

1. **Provide and maintain a Nix compilation target for Gleam.** That is, you should be able to write Gleam code and use it with Nix, aiming to **improve the Nix development experience** thanks to Gleam's **compile-time guarantees, type-safety and great tooling**, which should also **improve the accessibility and lower the entry barrier of working with Nix** (especially when it is needed to create complex and dynamic Nix configurations).

2. **Integrate the Gleam ecosystem with the Nix ecosystem as much as possible.** In particular, the existing [packages for Gleam](https://packages.gleam.run) should be made usable within Nix where possible.

In addition, we have the following **secondary goals**:

1. **Aid proper usage of the Gleam programming language among Nix users.** The tools we are creating to use Glistix within Nix, such as builders for packages, should ideally be easily reusable for the upstream Gleam compiler as well, eventually including proper support for the Erlang and JavaScript targets, if possible.

2. **Provide packages and bindings to aid in interacting with the Nix ecosystem from projects using Glistix.** For example, we'd like to release bindings for the easy creation of NixOS modules, configuration of flakes etc. with pure Gleam, and maybe even integrate some of those into the compiler itself. Our first step so far has been the creation of [the `glistix_nix`  package](https://github.com/glistix/nix).

3. **Contribute back to the upstream Gleam compiler where possible.** We have certain needs and challenges which are shared with Gleam users which use the Erlang and JavaScript targets. Therefore, **where possible, we'd like to contribute our solutions to upstream** and use them, in order to ensure we maintain some amount of uniformity with the rest of the Gleam ecosystem. This doesn't mean we can't add features exclusive to Glistix, but, **if they aren't Nix-related, they should be kept to a minimum**.

Finally, we want to clarify what **we are NOT trying to achieve:**

1. **Glistix is not meant to replace or compete with Gleam.** Rather, we are focused on **extending and integrating Gleam to the Nix ecosystem.** This includes the addition of the Nix target, but also the maintenance of tools to improve the Gleam on Nix experience as much as possible, as outlined above.

2. **Glistix is not meant to replace all usage of the Nix language.** We do want to provide a **better experience** when working with Nix, but it is absolutely not our goal (nor is it practical) to have Nix users switch to using exclusively Glistix for their configurations and other applications of Nix. This is also because **Nix's** (and `nixpkgs`) **APIs are large and change relatively often**, and thus it is not possible to always provide fully accurate type bindings for all aspects of the Nix ecosystem.

## Roadmap

Here is a non-exhaustive list of tasks we want to eventually tackle within the Glistix project. This can change at any time.

- [ ] **Add more real-world examples** and generally **make the documentation more robust.**
- [ ] Add some form of **tail-call optimization.** This depends on changes to Nix itself to be fully practical, but there might be improvements we can make in the meantime.
- [ ] Fully decide the semantics of **Nix's lazy evaluation** when using Glistix. In particular, we should take a final stance on how discarded expressions behave in this regard by default.
- [ ] **Improve the package management and patching story.** We should cooperate with the upstream Gleam compiler to avoid future incompatibilities. Currently, our methods of overriding the Gleam standard library through Git submodules are quite manual and need replacement with proper Git dependencies together with requirement overriding.
- [ ] **Create a playground page in which you can compile Gleam to Nix online.** Would be nice to be able to give Glistix a quick try in your web browser (maybe even in the documentation)!
