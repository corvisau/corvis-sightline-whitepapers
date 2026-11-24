A panel for editing the shared constants that your template files declare. Templates in the model can
reference a constant, so one change here updates every template that uses it. See
[Tetra templates](/docs/sightline/usage/tetra/templates) for how templates work.

## Opening It

Select **Constants** in the toolbar of an open Tetra diagram. You can also run
**Tetra: Edit Constants** from the command palette with a Tetra model
open. The toolbar button is disabled in a read-only view, because the editor
writes to the model. The command opens the editor in a read-only view too, with
every control disabled.

The panel is per-model. Opening it again for the same model focuses
the panel you already have. Opening it while a second model is open gives
that model its own panel. Close it like any editor tab.

If no Tetra model is open, the command tells you so and does nothing.

In the browser, select **Constants** to open an editor tab in the web shell. The diagram keeps its unsaved changes while you use the editor. A reader token opens the editor with every editing control disabled. A standalone Tetra page opens the editor in another browser tab.

## Constants Editor File List

The panel shows a **File** dropdown that lists every template file in the model, including files that declare no constants. Below it, a flat table lists every constant in the selected file:

| Column | Meaning |
|---|---|
| Path | The constant's dot-separated path (`review.cadenceDays` for a nested value) |
| Value | The constant's current value, editable |

Each row also has a **Delete** button. The table shows one file's constants at a time. Switching the **File** dropdown
swaps the table to that file's constants. A file with no constants shows
**No constants in this file.** A model with no template files shows
**This model has no template files.**

## Editing and Saving

Edit a value in place. Then select **Save**. The write goes to the constants
of that file only. If the save fails, the panel shows the reason below the
buttons and keeps your rows.

A constant's edit has no per-entity undo. A change affects every
template that references the constant the next time the model loads.

## Adding a Constant

1. Select the file in the **File** dropdown. A file with no constants gets its `const:` block when you save the first constant.
2. Select **Add constant**. A new row appears at the bottom of the table.
3. Type the path in the **Path** field and the value in the **Value** field.
4. Select **Save**.

The panel refuses a path in three cases:

- The path is empty or has an empty part, such as `review..days`
- The path already exists in the file
- The path is the start of another path in the file, such as `review` beside `review.cadenceDays`

The row shows the reason, and **Save** stays unavailable until you correct the path.

You can change a path only until you save. To rename a saved constant, delete it and add a new one.

## Deleting a Constant

Select **Delete** on the row. Then select **Save**. Deleting every constant of a file removes the `const:` block from that file.

## Value Types

The **Value** field takes one YAML value, so a constant keeps its type when you save.

| You type | Saved as |
|---|---|
| `90` | A number |
| `true` | A true/false value |
| `hello` | Text |
| `'90'` | The text `90` |
| `[a, b]` | A list |
| Nothing | Empty text |

The panel refuses a value that holds keys of its own, such as `a: 1`. Use a dot path to nest a constant. Text that looks like another type, such as `007` or `yes`, appears in quotes, and **Save** writes it back unchanged.

## Relationship to Lamina

The Lamina Constants Editor works the same way.
