A panel for editing the shared constants that your template files declare. Templates in the model can
reference a constant, so one change here updates every template that uses it. See
[Tetra templates](/docs/sightline/usage/tetra/templates) for how templates work.

## Opening it

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

## What it lists

The panel shows a **File** dropdown that lists every template file
that declares constants. Below it, a flat table lists every
constant in the selected file:

| Column | Meaning |
|---|---|
| Path | The constant's dot-separated path (`review.cadenceDays` for a nested value) |
| Value | The constant's current value, editable |

The table shows one file's constants at a time. Switching the **File** dropdown
swaps the table to that file's constants.

## Editing and saving

Edit a value in place. Then select **Save**. The write goes to the constants
of that file only.

A constant's edit has no per-entity undo. A change affects every
template that references the constant the next time the model loads.

## Relationship to Lamina

The Lamina Constants Editor works the same way.
