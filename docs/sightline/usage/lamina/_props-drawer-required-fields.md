Every field row in the props drawer carries a required or optional marker
(the accent bar on the left of the row).

## Required and optional markers

The marker shows whether the entity requires the field:

| Entity | Label |
|---|---|
| Cause | optional |
| Event | required |
| Outcome | required |
| Control | required |

The **Category** and **Control Type** fields of a control are also required. Every other field
in the drawer is optional.

The drawer does not block saving a required-but-blank field, so you can still
save a Cause without a label.

## Selecting a different entity always shows a clean form

Selecting a different node on the canvas, or a different row in the Tabular
Editor, resets every field editor to the new entity's data. This applies to
root-level and drilled-in fields alike.
