The Requirements tab toolbar (the id/ver/title/rule grid) offers two
independent axes, `[Filter]` and `[Edit columns]`. They follow the same
pattern as the Assessment Worklist (see [Worklist Filtering, Grouping, and Slices](/docs/sightline/usage/metron/worklist-filtering)).
The Worklist's `[Group]`, `[Saved filters]`, and compliance-slice picker have
no equivalent on the Requirements tab.

## Filter (per-view, ad hoc)

The **`[Filter]`** toolbar button is the shared
popover component, the same one the Worklist uses:

- **`[Filter]` → Basic tab**: pick an attribute, an operator (`equals` or,
  for text fields, `contains`), and a value. Multiple conditions AND
  together. Falls back to read-only "too complex for Basic" if the query
  is not cleanly representable as a condition list (for example, one edited in
  Advanced).
- **`[Filter]` → Advanced tab**: the full query builder, editing the exact
  same query as raw pipeline steps. A **Show code** button reveals
  the same query as read-only text with a Copy button. A statement can be disabled with the round button at the right of its row. When every statement is disabled, the filter matches nothing. See [Disabling a statement](/docs/sightline/usage/shared/tabular-filtering#disabling-a-statement).

Filtering narrows which requirement rows the grid shows; it does not affect
the underlying package data.

### Available Basic-tab filter fields

| Field | Type | Notes |
|---|---|---|
| ID | string (equals/contains) | The requirement id. |
| Title | string (equals/contains) | The requirement title. |
| Category | string (equals/contains) | The requirement's category path, joined with `/`. |
| Assessment method | enum (Manual / Rule) | How the requirement is assessed. |

The Basic tab does not offer the Rule that scopes a requirement as a field.
The Requirements grid's `rule` column already shows the title of that Rule; see
the "One Rule catalog, two reference points" section of the [Rules](/docs/sightline/usage/metron/rules) page.

## Column visibility

Column visibility is a separate axis from filtering. It controls the grid's
id/ver/title/rule columns, not which rows are shown.

The **`[Edit columns]`** toolbar button (the same column picker as in the Worklist and in the Tabular Editors of Lamina and Tetra) lets
you show/hide/reorder columns:

- Checking/unchecking a column in the popover shows/hides it in the grid
  immediately.
- The up/down arrows next to a visible column reorder it.
- Drag the handle at the left of a column header to move that column. To use the keyboard, focus the handle, press Space, press the left or right arrow key, then press Space again. The picker shows the same order, and a drag never sorts the column.
- All 4 columns are visible by default. The `rule` column shows the title of
  the Rule that scopes the requirement (see [Rules](/docs/sightline/usage/metron/rules)).
- Your choice is personal and local. It is stored in your own VS Code. It is
  independent of the column choice in the Worklist, Lamina or Tetra.
  The choice is not saved with the package.
- Dragging a column's resize handle saves its width the same way. Reopening the panel
  restores the resized width instead of the default.
