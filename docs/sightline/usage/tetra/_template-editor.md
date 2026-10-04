A panel for authoring the templates your entities inherit from, so you can edit
`*.template.yaml` files without opening them by hand. The YAML side of templates
is covered in the [Tetra templates](/docs/sightline/usage/tetra/templates) page.

## Opening it

Command palette -> **Tetra: Edit Templates**, with a `.tetra.yaml` model open.

The panel is per-model. Invoking the command again for the same model focuses
the panel you already have. Invoking it while a second model is open gives
that model its own panel. Close it like any editor tab.

If no Tetra model is open, the command tells you so and does nothing.

## What it lists

Every template declared across every file in the manifest's `templates:` list,
one row per template:

| Column | Meaning |
|---|---|
| id | The template id (`tpl-device-plc`) |
| label | Its `label:` field, when it has one |
| type | The entity type the template applies to |
| source file | The `*.template.yaml` file it is authored in |

Two dropdowns narrow the list: **File** (one declared template file, or all of
them) and **Type** (one entity type, or all of them).

### How the panel types a template

A template's type comes from the top-level YAML key it is authored under.
An entry in `devices:` is a device template, one in `zones:` a zone template.
The eight templatable types are the same eight that the [Tetra templates](/docs/sightline/usage/tetra/templates) page lists: zone,
group, device, network, information, channel, flow, system.

The id is the fallback. When the panel cannot read the key an entry sits under,
it parses the id as `tpl-<type>-<slug>` instead (`tpl-device-plc`; the plural
form `tpl-devices-plc` works too). That form is still the recommended way to
name a template, and it is what the panel mints for templates you add here.
A template named any other way, such as a bare `tpl-plc` under `devices:`,
is listed and edited exactly like the rest.

A template is left out of the panel only when neither answers. Its entry
sits under a key that is not one of the eight, and its id does not parse.
Such a template still works everywhere else: the loader resolves any id for
`template:` inheritance and for `{{> tpl-plc.field}}` partials. Move it under
the right key, or rename it to `tpl-<type>-<slug>`, to edit it here.

## Editing a template

Click a row to open the props drawer for that template. The form is the
edit schema for the row's own type, so a device template offers device fields
and a zone template offers zone fields.

A channel or flow template's `nodeA`/`nodeB` render the same node picker the
canvas gives channels and flows. It is a searchable dropdown over the model's
existing nodes. A system template's `members` renders
a chip multi-select over the same nodes. Both pickers are closed: an id typed but not selected
from the dropdown is not committed.

A field edit in the drawer stays pending. Select **Close** to keep the edit pending and return to the list. Select **Save all** on the save bar at the bottom of the editor to write every pending edit. **Undo** and **Discard all** on the same bar revert pending edits.

Edits go back to the template file the row came from. The other declared
template files stay byte-for-byte unchanged, even when several are open in the list.

## Adding, deleting and reordering

- **+ New template** opens the props drawer in create mode, which is the
  same drawer that edits a row. Choose the type at the top of the form. Then
  author the id and the label. The id is seeded as `tpl-<type>-`, and you
  complete the rest (`tpl-device-vendor-plc`). It is checked against every
  template id already in use, so a clash is refused before the write. When more
  than one template file is declared, the drawer also asks which file to write
  to.
- While a part-filled new template is open, a click on a row or a filter change
  makes the drawer ask before it discards your input.
- **Delete** on a row first checks the model for entities that inherit from that
  template. If any do, the confirmation lists them and the button becomes
  **Force delete**. Deleting anyway leaves those entities with a `template:`
  ref that no longer resolves, which the loader reports as a warning.
- **Drag a row** to reorder templates inside a file. Reordering rewrites the
  YAML order, so it is only available once you have narrowed to a single file
  and a single type.

## Managing template files

The toolbar's file controls change the manifest's `templates:` list:

- **Import file…** picks an existing `.yaml` / `.yml` file inside the model
  directory and adds it to `templates:`.
- **Create file…** writes a new empty `*.template.yaml` and adds it.
- **Remove from manifest** (shown once you have selected a single file)
  de-registers that file. It does not delete the file from disk. The
  templates in it stop loading until you add it back.

Files outside the model directory are rejected, including via a symlink.

## Constants

Template files may carry a top-level `const:` block. Tetra loads and applies
those constants. For the editor, see the
[Tetra Constants Editor](/docs/sightline/usage/tetra/constants-editor) page.
