A panel for editing the `const:` blocks your template files declare, so you can
change a shared constant without hand-editing YAML. The page
[Tetra templates — inheritance and Handlebars](/docs/sightline/usage/tetra/templates) covers the YAML side of constants.

## Opening it

Command palette -> **Tetra: Edit Constants**, with a `.tetra.yaml` model open.

The panel is per-model. Invoking the command again for the same model focuses
the panel you already have. Invoking it while a second model is open gives
that model its own panel. Close it like any editor tab.

If no Tetra model is open, the command tells you so and does nothing.

## What it lists

The panel shows a **File** dropdown that lists every declared template file
contributing a top-level `const:` block. Below it, a flat table lists every
constant in the selected file:

| Column | Meaning |
|---|---|
| Path | The constant's dot-separated path (`review.cadenceDays` for a nested value) |
| Value | The constant's current value, editable |

The table shows one file's constants at a time. Switching the **File** dropdown
swaps the table to that file's constants.

## Editing and saving

Edit a value in place, then **Save**. The write goes to that file's `const:`
block only. Every other declared template file, and every other file
carrying its own `const:` block, is left byte-for-byte unchanged.

A constant's edit has no per-entity undo. Any template in the model can
reference a `const:` value through Handlebars. A change affects every
template that references it the next time the model loads.

## Relationship to Lamina

The Lamina Constants Editor works the same way.
