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
