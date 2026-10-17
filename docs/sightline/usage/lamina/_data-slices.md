A **data slice** pairs a bonsai filter expression with an aggregation profile, giving the toolbar's picker a saved, named view of the model. This page covers the on-disk schema and the New/Edit/Duplicate/Delete authoring flow.

---

## DataSlice schema

A Lamina `DataSlice` has these fields:

| Field | Type | Notes |
|---|---|---|
| `id` | `string` | Opaque generated id (`s-xxxxx`) for New/Duplicate; unchanged when editing an existing slice |
| `name` | `string?` | Human-readable label shown in the toolbar's dropdown |
| `label` | `string?` | Deprecated alias for `name` |
| `filter` | `string \| string[] \| { program: string[] }` | Inline bonsai filter expression(s); array entries are AND-joined. An empty filter (`[]` or an empty string) selects the whole model, orphans included |
| `aggregationId` | `string` | Reference into `aggregations[].id`; `no-agg` is reserved for filter-only slices |
| `description` | `string?` | Optional prose description |
| `annotationDisplay` | `'full' \| 'id' \| 'hidden'` (optional) | How annotations render when this slice is active |
| `compact` / `showTags` / `showElements` | `boolean?` | Live view flags persisted with the slice |
| `outcomeSort` | optional | Persisted outcome sort order |

The loader rejects unknown keys in a data slice.

## Where a slice applies

A data slice and its aggregation change the diagram only. The **Data slice** and **Aggregation** pickers appear in the Diagram view, and they keep their selection while the Table view is open.

Lamina draws the diagram through a data slice. When a model has slices, one is always active: the slice you last used, or the first slice when that one no longer exists. Selecting a slice also selects its aggregation and applies its saved view settings. To view the whole model, create a slice with no filter.

A model with no data slices shows no **Data slice** control. The Diagram view shows the message "This model has no data slices. Create one to draw it." and a **Create data slice** button, which opens the slice editor. The Table view lists the model as usual.

A user who cannot edit the model sees the message "The model author hasn't created a data slice yet." instead, with no button. A web shell tab opened with a reader token is one example.

The Tabular Editor ignores the active slice. It lists every entity in the model, including staged creates and the entities that the **Orphans** view option hides from the canvas.

## Staged edits and the active slice

On the canvas, the active slice filters a staged (unsaved) create, delete, or control edit exactly like a saved entity. If the filter would exclude an entity, a staged create of that entity does not appear on the canvas. It appears once you switch to a slice whose filter includes it. The Tabular Editor lists the staged create at once. This applies to both filter forms (`filter: <bonsai expression>` and `filter: { program: [...] }`).

Staged field edits are the one exception. Editing a field that the active slice filters on does not re-evaluate the slice while you type. The slice re-evaluates once you save the edit.

## Creating, editing, and duplicating slices

Click the New/Edit/Duplicate/Delete icon buttons next to the toolbar's **Data slice** picker:

- **New** opens an empty editor. A slice saved with no filter lines has no filter and shows the whole model.
- **Edit** opens the currently-active slice with every field pre-filled; its id never changes.
- **Duplicate** opens the same editor pre-filled from the currently-active slice (name, filter, aggregation, description) but under a fresh id. Saving creates an independent copy and leaves the source slice unchanged.
- **Delete** asks for confirmation, then removes the active slice. The first remaining slice becomes active. When you delete the last slice, the confirmation warns that the diagram stays empty until you create another one.

A new or duplicated slice appears in the picker and becomes active as soon as you save the editor. It reaches the model file with your other unsaved changes when you click **Save all**.

In the web shell, the active data slice and the active aggregation belong to the browser tab. They survive a reload and are lost when you close the tab, so a new tab opens on the model's first slice. The VS Code extension keeps both choices per workspace.

Slice ids are opaque, generated identifiers (`s-` followed by 5 lowercase alphanumeric characters). They are not derived from the typed name.

The editor shows the slice's query as read-only text, with a line above it that reads **(no query)**, **1 statement** or the number of statements. Select **Edit query** to open the query builder in its own dialog. **Apply** returns the edited query to the editor, and **Cancel** or Escape keeps the query as it was. Escape closes only the query dialog.

A new slice has no query, and a slice with no query shows the whole model. **Edit query** on a slice with no query opens the builder on `model |> add()`, because a statement counts toward the result only when it ends in `add()`. The slice stays without a query until you select **Apply**.

The query builder, opened from **Edit query**, has a **Show code** button that reveals the exact bonsai program as read-only text, with a Copy button.

## Exporting the diagram

The **PNG** and **Draw.io** exports capture the diagram. On a model with no data slices they are disabled until you create one. Reports stay available.

The toolbar's Export menu can also write one PNG for each saved slice. It appears once the model file holds a slice, so a slice that is not yet saved does not count. See [PNG export](/docs/sightline/usage/lamina/png-export) for the file names, the framing and what the batch reads from disk.
