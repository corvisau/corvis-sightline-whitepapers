Three separate mechanisms narrow or organise what the assessment worklist shows. They look similar, since all live in the worklist toolbar, but they answer different questions. All three are view-only and change nothing that is assessed.

## Compliance slice (coarse, model-authored)

A **Compliance slice** is a compound filter authored on the assessment itself. It combines a category prefix, a set of tags, target entity kinds, and/or a Bonsai `entityFilter` program. It narrows the worklist to the pairs whose requirement and entity both satisfy the slice's ANDed predicates. Slices are a fixed, authored list per assessment. Picking one from the **Compliance slice** dropdown is a coarse, occasional choice.

The worklist has no Tetra-authored data slice picker. A pair exists only after it passes the `scopeRuleRef`-referenced Rule of its own requirement, so a Tetra data slice would narrow nothing further.

### Creating and editing Compliance slices

Use the New, Edit, Duplicate and Delete icon buttons next to the **Compliance slice** picker to author slices directly, instead of hand-editing the assessment YAML:

- **New** opens an empty editor. **Edit** opens the currently active slice; its id cannot change once created, so delete and recreate the slice instead. **Duplicate** opens the editor pre-filled from the active slice, with the id field seeded with a `<source-id>-copy` suggestion. The id stays editable and the existing collision guard still validates it. Saving creates an independent slice and leaves the source untouched. **Delete** asks for confirmation, then removes the slice. Deleting the active slice restores the worklist to "None" (the full list).
- The editor has six fields. Four are predicate fields: Category, Tags, Target kinds and Entity filter. They are optional and, when present, ANDed together:
  - **Slice id**: a stable identifier, unique among your authored slices.
  - **Name**: the display label shown in the picker.
  - **Category**: a `/`-separated prefix path (for example `Asset Management/Inventory`) matched against each requirement's own `category` path. A slice with `Asset Management` matches any requirement whose category starts with that segment.
  - **Tags**: key and optional value pairs matched against each requirement's resolved `attributes`, inherited from an attached `AttributeSet` or declared inline. A predicate with no value matches any value for that key.
  - **Target kinds**: restricts the slice to requirements whose scope targets any of the ticked entity kinds.
  - **Entity filter**: a Bonsai program that narrows which entities survive, independent of the requirement-level predicates above. It uses the same visual query builder as the Filter popover's Advanced tab. A **Show code** button reveals the exact program as read-only text with a Copy button.
- When no Compliance slice has been authored yet, the picker shows a synthetic "Default" placeholder (matches everything, mirrors "None"). It is never editable or deletable, and **New** replaces it with your first real slice.

### Assessing a progression together

A **progression** is a set of practices climbing MIL1&rarr;MIL3 for one progression area. For example, the AESCSF practices `ASSET-1a`, `1d` and `1f` all maintain an asset inventory at increasing maturity. The mechanisms above assess a progression together when composed:

1. Tag every practice in the progression with a shared `attributes` key, e.g. `{key: progression-area, value: asset-inventory}`.
2. Author a Compliance slice with `Tags: progression-area = asset-inventory` and select it. The worklist narrows to that progression's practices.
3. Click the worklist's header checkbox to select every currently visible row.
4. Right-click a selected row -> **Bulk edit (N pairs)** (see [Bulk edit](/docs/sightline/usage/shared/bulk-edit)) to apply an Outcome/State/Notes value across the whole progression in one action.

Each practice's outcome stays independently stored. Bulk edit applies the same value to many pairs at once and creates no link between them. A practice that needs a different outcome from its progression siblings is edited individually afterward, same as any other pair.

If practices in the progression also carry assessable attributes (see [Per-attribute assessment](/docs/sightline/usage/metron/attribute-assessment)), those are assessed per practice in the Assess drawer. Per-attribute state is deliberately excluded from bulk edit, so progression-wide bulk edit only ever touches Outcome/State/Notes.

## Filter + Group (per-view, ad hoc)

The `[Filter]` and `[Group]` toolbar buttons choose which of the pairs in scope to see right now and how to organise them. They are the shared popover components, used the same way across Metron, Lamina and Tetra. Both operate on the same underlying Bonsai `program`:

- **`[Filter]` → Basic tab**: pick an attribute, an operator (`equals` or, for text fields, `contains`), and a value. Multiple conditions AND together. If the program cannot be shown as a condition list, the Basic tab shows "too complex for Basic" and falls back to read-only. Examples are a program hand-edited in Advanced and one that references `follow()`.
- **`[Filter]` → Advanced tab**: the full query builder, editing the exact same Bonsai program as raw pipeline steps. Any expression Bonsai supports works here, including fields the Basic tab offers no picker for (see below). A **Show code** button reveals the same program as read-only text with a Copy button.
- **`[Group]`**: pick a low-cardinality attribute to group the grid rows by (or "None" to ungroup). Only a handful of fields are offered. Grouping by a near-unique field like Requirement or Entity would produce one row per group, which isn't useful.

Filter+Group replaces the worklist's old state chips (draft/in-progress/done/failed/all), the needs-revalidation toggle, the free-text "Filter pairs..." search box, and the per-column Req/Entity filter inputs.

### Available Basic-tab filter fields

| Field | Type | Notes |
|---|---|---|
| State | enum (draft / in-progress / done) | The pair's fundamental lifecycle state: the old state chips' values. |
| Needs revalidation | boolean | The pair's drift flag, replacing the old "needs revalidation" toggle. Also visible directly as the worklist's Reval column (see [Assessment overlay model](/docs/sightline/usage/metron/assessment-overlay-model)), with a right-click "Revalidate" action to clear it in bulk. |
| Outcome | enum (yes / no / partial / indeterminate / not applicable) | The fundamental outcome, which is the canonical compliance signal, rather than the raw scheme-specific outcome id. No derived pass/fail classification sits on top of it. Filter on the fundamental outcome you need, e.g. `Outcome = no OR Outcome = partial`. |
| Requirement | string (equals/contains) | The requirement id; replaces the old per-column Req filter. |
| Entity | string (equals/contains) | The entity id; replaces the old per-column Entity filter. |
| Notes | string (equals/contains) | Free-text search over the pair's notes; replaces the old global search box. |
| Category | string (equals/contains) | The pair's requirement's `category` path, joined with `/`. |

The Basic tab offers no field for a requirement's `attributes` (MIL/security-level/SLT), because they are a list of key/value pairs and not a single scalar value. The Advanced tab reaches them by direct path (`item.attributes.mil.value == "MIL3"`). On the worklist that path resolves the pair's full attribute map, not only what was typed inline. The map holds what the requirement inherits, what it declares, and anything the target state overrides for that one entity. ⛔ Never pipe an attribute path into `hasAny`/`hasAll`/`hasNone`. They return `false` for a record with no diagnostic, so the filter silently matches nothing. `tags` is an array, so `item.tags |> hasAny('EDR')` is correct there. See [Attributes](/docs/sightline/usage/metron/attributes).

### Available Group-by fields

Only **State** and **Outcome** are groupable. Each maps to a real worklist column with a low-cardinality value.

### Dashboard drill-in

Clicking a requirement row or a coverage segment on the Dashboard tab jumps to the worklist with a matching condition already applied (e.g. clicking the "draft" segment for ACM-1 sets `State = draft AND Requirement = ACM-1`). A drill-in replaces whatever filter program was active.

The Dashboard tab itself shows a distribution at every level (requirement, category, package root): a literal count of `done`-fundamental pairs per fundamental outcome (`yes`/`no`/`partial`/`indeterminate`/`not-applicable`). The fundamental outcome is the only compliance signal, so no scoring scheme, weighting or rollup method exists to configure.

## Saved Filters (personal, local)

The `[Saved filters]` button names and saves the current filter program so it can be re-applied or deleted later. Saved filters are:

- **Personal**: stored in your own VS Code, not in the model. Each toolbar has its own list, so Metron's worklist is independent of the saved filters of Lamina and Tetra.
- **Never part of the model**: not written to any `*.assessment.yaml` binding and not visible to other users who open the same package.
- **Named once**: no rename or edit affordance exists. To change a saved filter, save a new one, or delete it and save it again under the program you want.

For example, save `Outcome = no AND State = done` as "Non-compliant and done" to reapply it with one click next time you work through the backlog.

## Column visibility

The `[Edit columns]` toolbar button lets you show, hide and reorder the grid's Req/Entity/Outcome/State/Notes columns. It is the same column picker that Lamina and Tetra offer in their Tabular Editors. This is a different axis from the row-narrowing mechanisms above.

- Checking/unchecking a column in the popover shows/hides it in the grid immediately.
- The up/down arrows next to a visible column reorder it.
- Drag the handle at the left of a column header to move that column. To use the keyboard, focus the handle, press Space, press the left or right arrow key, then press Space again. The picker shows the same order, and a drag never sorts the column.
- The 6 original columns (Req ID, Req Text, Entity, Outcome, State, Notes) are visible by default. 7 more columns, joined from the pair's requirement, are selectable in the popover's "Requirement" group but hidden by default. They are version, category, description, tags, target entity kinds, scope filter summary, and assessment method. All 7 are read-only: they're a joined view of the requirement, not an edit surface for it (edit the requirement itself from the Requirements tab).
- The **Req** column shows `<requirement id> - <requirement title>` when the requirement is known, and the bare id otherwise. It shows the requirement's text as well as its id.
- Your choice is personal and local. It is stored in your own VS Code, independent of the column choice in Lamina or Tetra, and it is not written to any `*.assessment.yaml` binding.
- Dragging a column's resize handle saves its width the same way. Reopening the panel restores the resized width.
