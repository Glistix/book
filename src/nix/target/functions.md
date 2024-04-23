# Functions

We follow the conventions below for functions. Please keep those in mind when calling Gleam functions from Nix.

1. **Gleam functions with zero arguments are called with empty attribute sets.** For example, a function such as below would be called with `main { }`, which would give you the Nix integer `5` (as per ["Types"](./types.md)).
    ```gleam
    pub fn main() {
      5
    }
    ```

2. **Gleam functions with one or more arguments take them positionally.** For example, the function below would be called as `add 1 2` and would return `3`.
    ```gleam
    pub fn add(a: Int, b: Int) -> Int {
        a + b
    }
    ```

## Function bodies

Function bodies, just like blocks, are translated into `let...in` expressions. For example, the module below:

```gleam
pub fn myfunc() -> Int {
  let x = 5
  let y = 10
  let z = x * y
  let w = x - z

  x + y * w
}
```

is translated to

```nix
let
  myfunc = { }: let x = 5; y = 10; z = x * y; w = x - z; in x + (y * w);
in
{ inherit myfunc; }
```
