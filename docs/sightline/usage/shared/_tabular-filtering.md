The Tabular Editor grid has several mechanisms that narrow, organise or reshape what the grid shows. They all live in the tabular toolbar and answer different questions. None of them changes the underlying model.

Tetra has five mechanisms, and Lamina has four. The Tetra-only mechanism is the column header filter. Tetra also has extra columns, address operators and a second group-by field. The [Tetra](#tetra) and [Lamina](#lamina) sections give the per-app detail.

The Metron assessment worklist has the analogous **Filter**, **Group** and **Saved filters** toolbar over a different, scheme-driven field set. See [Worklist filtering](/docs/sightline/usage/metron/worklist-filtering).

A data slice is a coarser mechanism that the model author defines. A data slice narrows the whole model, on both the canvas and the Tabular Editor. **Filter** and **Group** reshape only the grid, on top of whichever data slice is active. For Tetra, see [Data slice filtering](/docs/sightline/usage/tetra/data-slice-filtering). For Lamina, see [Lamina Data Slices](/docs/sightline/usage/lamina/data-slices).

---

## Entity type filter

The entity type filter selects which entity kinds the grid shows. Lamina offers a row of chips with **all** and **none** links. Tetra offers no chips and narrows by type through the filter button on the `type` column header.

The entity type filter composes (AND) with the **Filter** popover. The available types and the default differ by app. See the [Tetra](#tetra) and [Lamina](#lamina) sections.

## Filter and Group

The **Filter** and **Group** toolbar buttons open the shared query popovers. Lamina, Metron and Tetra use them in the same way. They operate on the same underlying Bonsai program.

- **Filter**, Basic tab: pick an attribute, an operator and a value. Multiple conditions AND together. Which operators are offered depends on the field type (see the table below). Sometimes the program is not cleanly representable as a condition list, for example after someone edits it by hand in the Advanced tab. Then the Basic tab shows "too complex for Basic" and becomes read-only.
- **Filter**, Advanced tab: the full query builder, which edits the exact same Bonsai program as raw pipeline steps. Anything that Bonsai can express is allowed, including `follow()` traversals. The traversal names differ by app. A **Show code** button reveals the same program as read-only text with a Copy button.
- **Group**: pick a low-cardinality attribute to group the grid rows by, or choose "None" to remove the grouping.

The grid has no per-column text boxes under the header. Filter and Group replace them.

### Available Basic-tab filter fields

Every filterable column is a Basic-tab field. The column set defines the fields. Operators follow the field type.

| Field type | Operators |
|---|---|
| Text (`id`, `label`, `description`, per-type string fields) | equals, contains, is empty |
| List (`tags`, per-type string-array fields) | has any, is empty |
| Enum, boolean | equals, is empty |

Per-type fields are Basic-tab fields as well as the fields common to every entity type. Tetra adds one more field type, Address. Lamina lists `zones` as a list field and names further examples. See the [Tetra](#tetra) and [Lamina](#lamina) sections.

The `tags` field uses the `has any` operator (Bonsai `hasAny`). The `contains` operator does not match list members.

### Available group-by fields

Group offers only low-cardinality attributes. ID, Label and Description are near-unique per row, so Group does not offer them. The entity type filter narrows by entity kind, and Group organises the rows within whichever narrowing is active. The fields that Group offers differ by app.

## Saved filters

In Lamina and Metron, the **Saved filters** button names and saves the current filter program. In Tetra, **Saved filters** is a section at the bottom of the **Filter** popover, and there is no separate button. You can re-apply or delete a saved filter later. Saved filters have these properties:

- Personal. The app stores them in your own VS Code, scoped to this toolbar. Each app keeps its own list, independent of the lists in the other apps.
- Never part of the model. The app writes them to no model YAML file and does not share them with other users.
- Named once. There is no rename or edit action. To change a saved filter, save a new one, or delete and re-save it, under the program you want.

The storage key for each app is in the [Tetra](#tetra) and [Lamina](#lamina) sections.

## Column visibility

The **Edit columns** toolbar button (**Columns** in Tetra, at the right of the table header) opens a popover that shows, hides and reorders columns. This is a different axis from the row-narrowing mechanisms above: it controls which of the available columns render. Every Metron grid with an **Edit columns** button has the same picker.

- Checking or unchecking a column in the popover shows or hides it in the grid immediately.
- Column order and visibility are personal and local. The app stores them in your own VS Code, independent of the other apps, and writes them to no model YAML file.
- Dragging the handle at the left of a column header moves that column. To use the keyboard, focus the handle, press Space to pick the column up, press the left or right arrow key to move it, and press Space again to drop it. The popover list shows the same order, and the app saves it the same way. Dragging a header never sorts the column. In Metron, the always-on columns that the picker does not list (such as the row actions) have no handle and stay after the others.
- Dragging a column resize handle saves its width the same way. Reopening the panel restores the resized width instead of the default.
- The column set never changes with the entity type filter, Filter or Group. A cell of a row type that does not declare a given field renders blank.

The storage keys for each app are in the [Tetra](#tetra) and [Lamina](#lamina) sections.

---

## Tetra

Tetra has five mechanisms: Filter, Group, the column header filter, and column visibility with Saved filters, plus the Tree view, which changes how the grid lays out the rows that remain.

In the Table view, **Columns**, **Filter**, **Tree** and **Group** sit at the far right of the table header row. They stay in view when the table scrolls. **New** sits in the top toolbar, in place of the **View** menu. The **View** menu and the diagram **New** menu show only in the Diagram view.

### Entity type

The table opens with every entity type visible: the eight entity types plus `annotation`. No chips show. To narrow the rows by type, open the filter button on the `type` column header and untick the types that you do not want. **(Select All)** restores every type.

### Source file

A Tetra model can span several `.tetra.yaml` files. The grid shows the rows of every file by default, and no toolbar button selects files. The `sourceFile` column filters like any other column. Open its header filter, or add a `sourceFile` condition in **Filter** > Basic, to keep only some files.

The `sourceFile` popout opens with **(Blanks)** ticked. An entity that you created and have not yet saved to a file has no source file, and it stays in the grid while you untick files. Untick **(Blanks)** to hide those rows. Like every program-backed header filter, the `sourceFile` button leaves the header while the Advanced tab holds a hand-edited query. Filter on `item.sourceFile` in the query instead.

The `sourceFile` condition composes (AND) with the `type` column filter and with every other condition of **Filter**. Lamina has no equivalent, because its models are always single-file.

### Advanced tab traversals

The Advanced tab supports `follow('parent')`, `follow('children')`, `follow('nodeA')`, `follow('nodeB')` and `follow('path')`. They traverse the containment tree and the flow and channel endpoints.

### Column header filter

Every filterable column header carries a filter button. The button opens an Excel-style popout. The popout has these controls:

- Sort ascending and descending.
- A value search box.
- A checklist of the distinct values in the column, with **(Select All)** and **(Blanks)**.
- A `contains` or `equals` condition.

The popout writes into the same Bonsai program that the **Filter** popover edits. A header filter and a Basic-tab condition are one object. A filter that you set from the header appears in **Filter** > Basic as an editable condition. If you remove it there, the ticks in the header popout clear. **Filter** stays the only place that shows the conditions across all columns at once, and the only route to the Advanced tab.

- The checklist lists the values that survive every other column's filter, not the filter of this column. Ticking two values therefore never collapses the list to those two.
- **(Blanks)** also matches rows whose type has no such field. A schema column renders empty both when the field is empty and when the entity type of the row does not declare the field. The tooltip of the checkbox says so.
- Above 200 distinct values, the checklist is suppressed. A list of thousands is no use on a near-unique column such as `id`. The search box, the condition and **(Blanks)** stay available.

The `type` column is the exception. Its header popout selects the entity types instead of writing to the program.

The `value`, `linked to` and `addressing` columns have no filter button. Their cells show a flattened summary of a compound field (annotation values, annotation links, network address ranges). A chosen summary string has no equivalent condition on the underlying field.

The header filter is available only while the program is Basic-representable. After you edit the query by hand in the Advanced tab, the header buttons leave the program-backed columns, because the app cannot rewrite the part of the query for one column without discarding your own Bonsai.

A multi-value condition that you set from a column header (`is one of`, `has any`) shows its values as removable chips. Remove one chip to narrow the condition, and remove the last chip to drop the condition. Build such a condition from the header popout. The Basic tab is where you see and prune it.

### Tree view

The **Tree** button switches the grid between the flat list and a tree. The tree nests each row under the node that contains it, so a zone shows its groups, devices and networks beneath it. A chevron on a row expands and collapses its children. Every column, inline edit and row action works in the tree as it does in the flat list. Sorting orders the rows within each parent.

The tree opens with its first level expanded. Expansion resets when you leave the tree. The grid remembers whether you chose the tree or the flat list, and restores that choice the next time you open the table.

Two kinds of row have no containing node in the tree:

- A row whose containing node is not in the model, such as a device in the orphaned devices group, shows at the top level.
- Channels, flows, systems and annotations have no containing node. Each type that has rows sits under one folder row at the end of the tree, for example **Channels (12)**. A folder row is a label. It has no checkbox, no context menu and no editable cell.

A filter keeps the structure around each match. Suppose a row passes the type filter, a column filter or **Filter**, but the nodes that contain it do not. The grid still shows those nodes in muted italic text, and the tree expands to reveal every match. A muted row is a context row. It has no checkbox, and the select-all checkbox in the header skips it. A click on a context row opens its properties instead of selecting it. A context row still edits and still opens its context menu.

Selecting a parent row selects only that row. Collapsed children stay unselected, so bulk edit changes only the rows that you tick. A new entity that you have not saved appears under its parent, and the tree expands to show it.

The tree and **Group** exclude each other. Choosing **Tree** clears the grouping, and choosing a group returns the grid to the flat list.

### Address filter fields

Tetra adds a fourth field type to the Basic-tab table.

| Field type | Operators |
|---|---|
| Address (`addressing`) | covers, overlaps, is templated, is empty |

The `covers` and `overlaps` operators take one address or one CIDR range (for example `10.1.2.5` or `10.1.0.0/16`). They match networks whose `addressing` list holds a range that contains the value or shares address space with it. The `is templated` operator takes no value and matches networks that have a `{placeholder}` range. Elements that are not networks have no `addressing` list, so `covers` and `overlaps` never match them. See [Data slice filtering](/docs/sightline/usage/tetra/data-slice-filtering#address-semantic-transforms) for the transforms behind these operators.

The `addressing` column has no header filter, so build these conditions from the **Filter** Basic tab.

### Group-by fields

Tetra offers **Type** and **File**. The **Type** option organises rows by entity kind, and the **File** option organises them by source file, within whichever narrowing is active.

Three metadata columns are off by default. Enable them from the **Columns** popover.

- **parent** (group `Common`): the id of the immediate containing node of the row. The cell is blank for a top-level row and for `channel`, `flow`, `system` and `annotation` rows, which have no tree parent.
- **children** (group `Container`): the direct member ids of a `device`, `zone`, `group` or `network` row, comma-joined. The cell is blank for every other row type.
- **status** (group `Container`): `pruned`, `orphan` or `ok` for a `device`, `zone`, `group` or `network` row, shown as a coloured pill. The value is `pruned` when the `{ id, filter }` claim filter of the row excluded it. A pruned row stays editable and is hidden from the canvas by default. The value is `orphan` when the row is defined but unreachable from the diagram tree, and `ok` otherwise. The cell is blank for every other row type. The column is read-only. Edit the tree position of a row from the canvas or the props drawer, not from this column.

---

## Lamina

Lamina has four mechanisms: the entity type chips, Filter, Group, and column visibility with Saved filters.

### Entity type chips

The chip row at the left of the toolbar has the chips Causes, Events, Outcomes and Controls, plus **all** and **none** links. Metron has a compliance slice that folds into its **Filter**, but Lamina has no equivalent. The entity type chips therefore stay their own control.

### Advanced tab traversals

The Advanced tab supports `follow('causes')`, `follow('events')` and `follow('outcomes')`. They traverse the forward and reverse edges of the bow-tie graph.

### Filter fields

Per-type fields are Basic-tab fields. Examples are the `state` field of a cause, the `category` and `controlType` fields of a control, and the `materialRisk` field of an outcome. The `zones` field is a list field, like `tags`.

Two kinds of field are Advanced-only:

- `entityType`. The entity type chips already cover entity-kind filtering, so Lamina does not duplicate it as a Basic-tab field.
- Any compound field, such as the `actor`, `vector` and `exposure` fields of a cause, `controls`, or the `consequence` field of an outcome. A compound field has no scalar shape that a Basic-tab condition can express.

Both stay reachable from the Advanced tab, for example with `item.state |> hasAny('...')` for the fields that apply.

### Group-by fields

Lamina offers only **Type** (cause, event, outcome or control). ID, Template, Label and Description are near-unique or high-cardinality per row, so Group does not offer them. Lamina models are always single-file, so there is no File option.
