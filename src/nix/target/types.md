# Types

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
    <td>bool (true, false)<br></td>
    <td>Full</td>
    <td></td>
  </tr>
  <tr>
    <td>Int</td>
    <td>Int</td>
    <td>Full*</td>
    <td>- Signed 64 bits (not unrestricted)<br>- "0b/0x/0o" literals parsed at runtime<br></td>
  </tr>
  <tr>
    <td>Float</td>
    <td>Float</td>
    <td>Full</td>
    <td>- Excessively large float literals can cause a runtime error</td>
  </tr>
  <tr>
    <td>String</td>
    <td>String</td>
    <td>Full</td>
    <td>- Unicode escapes parsed using TOML at runtime</td>
  </tr>
  <tr>
    <td>Functions</td>
    <td>Lambdas</td>
    <td>Full*<br></td>
    <td>- Cannot compare functions for equality (always false)<br>- Functions with zero arguments take a single {} argument<br>- Functions with one or more arguments are positional</td>
  </tr>
  <tr>
    <td>Tuples<br></td>
    <td>Arrays (Lists)</td>
    <td>Full</td>
    <td></td>
  </tr>
  <tr>
    <td>Lists</td>
    <td>Nested attribute sets<br></td>
    <td>Full<br></td>
    <td>- No tail call optimization, so traversing large lists can cause a stack overflow</td>
  </tr>
  <tr>
    <td>Records</td>
    <td>Tagged attribute sets</td>
    <td>Full</td>
    <td></td>
  </tr>
  <tr>
    <td>BitArray<br></td>
    <td>Attribute set with array of bytes</td>
    <td>Partial</td>
    <td>- Must be byte-aligned (can't specify individual bits)<br>- Limited support for specifying segments (sized integers, UTF-8 strings and other bit arrays)<br>   - Compared to the JS target, doesn't support float-&gt;bytes conversion<br>- Limited pattern matching support (equivalent to JS here)<br></td>
  </tr>
</tbody>
</table>
