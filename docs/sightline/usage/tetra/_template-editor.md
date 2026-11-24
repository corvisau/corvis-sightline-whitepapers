A panel for authoring the templates your entities inherit from. How templates
behave is covered in the [Tetra templates](/docs/sightline/usage/tetra/templates) page.

## Opening the Template Editor

Select **Templates** in the toolbar of an open Tetra diagram. You can also run
**Tetra: Edit Templates** from the command palette with a Tetra model
open. The toolbar button is disabled in a read-only view, because the editor
writes to the model. The command opens the editor in a read-only view too, with
every control disabled.

The panel is per-model. Opening it again for the same model focuses
the panel you already have. Opening it while a second model is open gives
that model its own panel. Close it like any editor tab.

If no Tetra model is open, the command tells you so and does nothing.

In the browser, select **Templates** to open an editor tab in the web shell. The diagram keeps its unsaved changes while you use the editor. A reader token opens the editor with every editing control disabled. A standalone Tetra page opens the editor in another browser tab.

## Template Editor File List

In the browser, **Import file…** asks for the path of an existing YAML file relative to the model directory on the server. This registers that file as a template file. **Create file…** creates a new file in the same model directory.

Every template in every template file the model registers, one row per
template. The rows sit in a grid. With **All types** selected, the grid shows
these columns:

| Column | Meaning |
|---|---|
| id | The template id (`tpl-device-plc`) |
| label | Its label, when it has one |
| type | The entity type the template applies to |
| source file | The template file it is authored in |

Two dropdowns narrow the list: **File** (one template file, or all of
them) and **Type** (one entity type, or all of them).

When you select one type, the grid shows that type's own fields as columns, after
the id and the label, with the source file last. A device template shows device
fields, and a zone template shows zone fields.

The grid header has **Columns**, **Filter** and **Group** controls. **Columns**
shows, hides and reorders columns. **Filter** narrows the rows by conditions.
**Group** groups the rows by a column. Each column header also has its own
filter. The grid remembers the columns you choose for each type.

The type is one of the eight templatable types that the [Tetra templates](/docs/sightline/usage/tetra/templates) page lists: zone,
group, device, network, information, channel, flow, system. The recommended id
form is `tpl-<type>-<slug>` (`tpl-device-plc`), and it is what the panel seeds
for templates you add here. A template named any other way is listed and
edited exactly like the rest. A template whose type the panel cannot work out
is left out of the list.

A model that registers no template file shows **No template files declared in
this manifest.** with **Import existing template file…** and **Create new
template file…**. They work as **Import file…** and **Create file…** below.

## Editing a Template

Click a row, outside a cell you can edit, to open the props drawer for that
template. Double-clicking such a cell, or pressing Enter in it, does the same. In
a cell you can edit, a click, a double-click or Enter edits the cell instead.
Right-click a row for its menu: **Open** first, then **Move up** and **Move
down** where the order can change, then **Delete**. The form is the
edit schema for the row's own type, so a device template offers device fields
and a zone template offers zone fields.

A channel or flow template's `nodeA` renders the same node picker the canvas
gives channels and flows. It is a searchable dropdown over the model's existing
nodes. Its `nodeB` renders the same target rows as the drawer. A system
template's `members` renders a chip multi-select over the same nodes. These
pickers are closed: an id typed but not selected from the dropdown is not
committed.

With one type selected, a cell for a text, number, true/false or choice field
edits in place. Click the cell, change the value, and press **Enter**. Press
**Escape** to cancel. A number field refuses text that is not a number. The id
and the source file are read-only. A field with its own editor, such as the
template parent, properties or links, opens in the drawer.

A field edit in the drawer or in a cell stays pending. Select **Close** to keep the edit pending and return to the list. Select **Save all** on the save bar at the bottom of the editor to write every pending edit. **Undo** and **Discard all** on the same bar revert pending edits.

Edits go back to the template file the row came from. The other template
files stay unchanged, even when several are open in the list.

## Adding, Deleting and Reordering

- **+ New template** opens the props drawer in create mode, which is the
  same drawer that edits a row. Choose the type at the top of the form. Then
  author the id and the label. The id is seeded as `tpl-<type>-`, and you
  complete the rest (`tpl-device-vendor-plc`). It is checked against every
  template id already in use, so a clash is refused before the write. When more
  than one template file is declared, the drawer also asks which file to write
  to. An optional template selector lists the templates of the chosen type.
  Pick one to create the new template as a sub-template of it, or leave it
  empty.
- A part-filled new template can be left open. A click on a row or a filter
  change then makes the drawer ask before it discards your input.
- **Delete** on a row first checks the model for entities that inherit from that
  template. If any do, the confirmation lists them and the button becomes
  **Force delete**. Deleting anyway leaves those entities pointing at a
  template that no longer exists, which Tetra reports as a warning.
- **Move up** and **Move down** on a row reorder templates inside a file.
  They appear once you have narrowed to a single file and a single type.

## Managing Template Files

The toolbar's file controls change which template files the model registers:

- **Import file…** picks an existing template file inside the model
  directory and registers it.
- **Create file…** opens the **Create new template file** dialog. Enter a file
  name relative to the model directory. **Create** stays disabled until the name
  is valid. The dialog creates an empty template file and registers it.
- **Remove from manifest** (shown once you have selected a single file)
  unregisters that file. It does not delete the file from disk. The
  templates in it stop loading until you register it again.

The editor rejects files outside the model directory.

## Constants

Template files can also carry constants that templates reference. For the
editor, see the [Tetra Constants Editor](/docs/sightline/usage/tetra/constants-editor) page.
