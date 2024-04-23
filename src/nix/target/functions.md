# Functions

We follow the convention below for functions. Please keep those in mind when calling Gleam functions from Nix.

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

