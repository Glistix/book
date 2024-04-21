# Types

## Built-in types

The table below shows how Glistix translates most common Gleam types into Nix types.

<table>
<thead>
  <tr>
    <th>Gleam Type</th>
    <th>Nix Type</th>
    <th>Support</th>
    <th>Notes</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>Bool</td>
    <td>Bool (<code>true</code>, <code>false</code>)</td>
    <td>Full</td>
    <td></td>
  </tr>
  <tr>
    <td>Int</td>
    <td>Int</td>
    <td>Full*</td>
    <td><ul><li>Signed 64 bits (not unrestricted)</li><li>"0b/0x/0o" literals parsed at runtime</li></ul></td>
  </tr>
  <tr>
    <td>Float</td>
    <td>Float</td>
    <td>Full</td>
    <td><ul><li>Excessively large float literals can cause a runtime error</li></ul></td>
  </tr>
  <tr>
    <td>String</td>
    <td>String</td>
    <td>Full</td>
    <td><ul><li>Unicode escapes parsed using TOML at runtime</li></ul></td>
  </tr>
  <tr>
    <td>Functions</td>
    <td>Lambdas</td>
    <td>Full*<br></td>
    <td><ul><li>Cannot compare functions for equality (always false)</li><li>Functions with zero arguments take a single <code>{ }</code> argument</li><li>Functions with one or more arguments take positional arguments</li></ul></td>
  </tr>
  <tr>
    <td>Tuples</td>
    <td>Arrays (Nix Lists)</td>
    <td>Full</td>
    <td></td>
  </tr>
  <tr>
    <td>Lists</td>
    <td>Nested attribute sets<br><br>When the tag is <code>Empty</code>, has no fields; when the tag is <code>NotEmpty</code>, has <code>head</code> (contained element) and <code>tail</code> (next item)<br></td>
    <td>Full<br></td>
    <td><ul><li>No tail call optimization, so traversing large lists can cause a stack overflow</li></ul></td>
  </tr>
  <tr>
    <td>Records</td>
    <td>Tagged attribute sets (see below)</td>
    <td>Full</td>
    <td></td>
  </tr>
  <tr>
    <td>Bit Arrays</td>
    <td>Attribute set with <code>buffer</code> field containing array of bytes (0-255 integers)</td>
    <td>Partial</td>
    <td><ul><li>Must be byte-aligned (can't specify individual bits)</li><li>Limited support for specifying segments (sized integers, UTF-8 strings and other bit arrays)</li><li>Compared to the JS target, doesn't support float-&gt;bytes conversion</li><li>Limited pattern matching support (equivalent to JS here)</li></ul></td>
  </tr>
</tbody>
</table>

## Records

User-created types are translated to Nix as follows:

1. **Types without constructors only exist in the Gleam type system.** Therefore, **it is not possible to create a type without a constructor** unless you do it **through FFI**. This is useful to create Gleam representations of Nix types. For example, you can define an `Array` type which you can't construct through Gleam, but can through FFI:

    ```gleam
    // Only constructible via FFI
    pub type Array(a)

    /// Create a new Array
    @external(nix, "./ffi.nix", "createArray")
    pub fn new() -> Array(a)
    ```

    Then, on the Nix side (`./ffi.nix`):

    ```nix
    let
      createArray = { }: [ ];
    in { inherit createArray; }
    ```

    Otherwise, the `Array` type **is not known to Nix at all** (even if it understands the underlying representation of instances of `Array`).

2. **Records with constructors are always represented by attribute sets.** Those attribute sets contain, **at least, their constructors' tags**, as well as **any fields**. For example:
    ```gleam
    pub type Example {
      Constructor1
      Constructor2(Int, Float)
      Constructor3(field: Int, inherit: Int)  // Nix keyword? No problem
      Constructor4(Int, mixed: Int, Float, Int)
    }
    ```

    The module above compiles to

    ```nix
    let
      # ! A record without fields is not a function !
      Constructor1 = { __gleamTag = "Constructor1"; };

      Constructor2 = x0: x1: { __gleamTag = "Constructor2"; _0 = x0; _1 = x1; };

      Constructor3 =
        field: inherit':
          { __gleamTag = "Constructor3"; inherit field; "inherit" = inherit'; };

      Constructor4 =
        x0: mixed: x2: x3:
          { __gleamTag = "Constructor4"; inherit mixed; _0 = x0; _2 = x2; _3 = x3; };
    in
    { inherit Constructor1 Constructor2 Constructor3 Constructor4; }
    ```

    Note that **positional fields become `_N` fields,** where `N` is the field's position. **Named fields keep their names, even if they are Nix keywords.**

    You can **construct these records in Nix** by just **calling their constructors.** In this case, `Constructor1` can be used directly; `Constructor2(a, b)` in Gleam would correspond to `Constructor2 a b` in Nix; and so on.
