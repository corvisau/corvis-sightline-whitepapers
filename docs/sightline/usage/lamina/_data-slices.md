A **data slice** pairs a bonsai filter expression with an aggregation profile, giving the toolbar's picker a saved, named view of the model. This page covers the on-disk schema and the New/Edit/Duplicate/Delete authoring flow.

---

## DataSlice schema

A Lamina `DataSlice` has these fields:

| Field | Type | Notes |
|---|---|---|
| `id` | `string` | Opaque generated id (`s-xxxxx`) for New/Duplicate; unchanged when editing an existing slice |
| `name` | `string?` | Human-readable label shown in the toolbar's dropdown |
| `label` | `string?` | Deprecated alias for `name`, kept for back-compat with slices persisted by older builds |
| `filter` | `string \| string[] \| { program: string[] }` | Inline bonsai filter expression(s); array entries are AND-joined |
| `aggregationId` | `string` | Reference into `aggregations[].id`; `no-agg` is reserved for filter-only slices |
| `description` | `string?` | Optional prose description |
| `annotationDisplay` | `'full' \| 'id' \| 'hidden'` (optional) | How annotations render when this slice is active |
| `compact` / `showTags` / `showElements` | `boolean?` | Live view flags persisted with the slice |
| `outcomeSort` | optional | Persisted outcome sort order |

The loader rejects unknown keys in a data slice.

## Staged edits and the active slice

The active slice filters a staged (unsaved) create, delete, or control edit exactly like a saved entity. If the filter would exclude an entity, a staged create of that entity does not appear on the canvas or in the Tabular Editor. It appears once you clear the slice, or switch to one whose filter includes it. This applies to both filter forms (`filter: <bonsai expression>` and `filter: { program: [...] }`).

Staged field edits are the one exception. Editing a field that the active slice filters on does not re-evaluate the slice while you type. The slice re-evaluates once you save the edit.

## Creating, editing, and duplicating slices

Click the New/Edit/Duplicate/Delete icon buttons next to the toolbar's **Data slice** picker:

- **New** opens an empty editor.
- **Edit** opens the currently-active slice with every field pre-filled; its id never changes.
- **Duplicate** opens the same editor pre-filled from the currently-active slice (name, filter, aggregation, description) but under a fresh id. Saving creates an independent copy and leaves the source slice unchanged.
- **Delete** asks for confirmation, then removes the active slice.

Slice ids are opaque, generated identifiers (`s-` followed by 5 lowercase alphanumeric characters). They are not derived from the typed name.

The editor's query builder has a **Show code** button that reveals the exact bonsai program as read-only text, with a Copy button.

## Exporting every slice

The toolbar's Export menu can write one PNG for each saved slice. See [PNG export](/docs/sightline/usage/lamina/png-export) for the file names, the framing and what the batch reads from disk.
