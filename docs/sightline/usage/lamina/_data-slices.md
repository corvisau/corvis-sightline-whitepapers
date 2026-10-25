A **data slice** pairs a bonsai filter expression with an optional aggregation profile, giving the toolbar's picker a saved, named view of the model. This page covers where a slice applies and the New, Edit, Duplicate and Delete flow.

---

## Where a slice applies

A data slice and its aggregation change the diagram only. The **Data slice** menu appears in the Diagram view, and the slice and the aggregation keep their selection while the Table view is open. The aggregation is chosen in the **Model Properties** drawer; see [Choosing an aggregation](#choosing-an-aggregation).

Lamina draws the diagram through a data slice. When a model has slices, one is always active: the slice you last used, or the first slice when that one no longer exists. Select a slice, and Lamina also selects its aggregation, if it has one, and applies its saved view settings. To view the whole model, create a slice with no filter.

A model with no data slices shows no **Data slice** control. The Diagram view shows the message "This model has no data slices. Create one to draw it." and a **Create data slice** button, which opens the slice editor. The Table view lists the model as usual.

A user who cannot edit the model sees the message "The model author hasn't created a data slice yet." instead, with no button. A web shell tab opened with a reader token is one example.

The Tabular Editor ignores the active slice. It lists every entity in the model, including staged creates and the entities that the **Orphans** view option hides from the canvas.

## Staged edits and the active slice

On the canvas, the active slice filters a staged (unsaved) create, delete, or control edit exactly like a saved entity. If the filter would exclude an entity, a staged create of that entity does not appear on the canvas. It appears once you switch to a slice whose filter includes it. The Tabular Editor lists the staged create at once.

Staged field edits are the one exception. When you edit a field that the active slice filters on, the slice does not re-evaluate while you type. The slice re-evaluates once you save the edit.

## Creating, editing, and duplicating slices

Open the **Data slice** menu in the toolbar. The button reads **Data slice:** and the name of the active slice. The menu lists the slices, and the **New**, **Edit**, **Duplicate** and **Delete** buttons sit in a row at the top of it:

- **New** opens an empty editor. A slice saved with no filter lines has no filter and shows the whole model.
- **Edit** opens the currently-active slice with every field pre-filled.
- **Duplicate** opens the same editor pre-filled from the currently-active slice (name, filter, aggregation, description) but as a new slice. When you save it, Lamina creates an independent copy and leaves the source slice unchanged.
- **Delete** asks for confirmation, then removes the active slice. The first remaining slice becomes active. When you delete the last slice, the confirmation warns that the diagram stays empty until you create another one.

A new or duplicated slice appears in the picker and becomes active as soon as you save the editor. Lamina saves it with your other unsaved changes when you click **Save all**. When you save an edit to the active slice, Lamina selects the slice's aggregation at once and keeps your view settings.

In the web shell, the active data slice and the active aggregation belong to the browser tab. They survive a reload, and you lose them when you close the tab, so a new tab opens on the model's first slice. The VS Code extension keeps both choices per workspace.

The editor has a **Name** field, a **Query** block, an **Aggregation** picker and a **Description** field. The **Aggregation** picker offers **None**, and a slice saved with **None** draws without an aggregation. **None** is the default when the model has no aggregations, so a new model can create its first slice with only a name. The picker also lists an aggregation you created since the last **Save all**. The editor shows the slice's query as read-only text, with a line above it that reads **(no query)**, **1 statement** or the number of statements. Select **Edit query** to open the query builder in its own dialog. **Apply** returns the edited query to the editor, and **Cancel** or Escape keeps the query as it was. Escape closes only the query dialog.

A new slice has no query, and a slice with no query shows the whole model. **Edit query** on a slice with no query opens the builder on `model |> add()`. A statement counts toward the result only when it ends in `add()`. The slice stays without a query until you select **Apply**.

The query builder, opened from **Edit query**, has a **Show code** button that reveals the exact bonsai program as read-only text, with a Copy button.

A statement can be disabled and enabled again with the round button at the right of its row. A slice whose statements are all disabled shows the whole model, as a slice with no query does. See [Disabling a statement](/docs/sightline/usage/shared/tabular-filtering#disabling-a-statement).

## Choosing an aggregation

Choose **Model Properties** in the toolbar. The **Aggregation** picker at the top of the drawer lists **None** and every aggregation of the model. A choice applies to the diagram at once. It does not count as an unsaved change, and **Save all** has nothing to write for it.

The add, edit and delete buttons beside the picker create, change and remove an aggregation. Edit and Delete act on the aggregation the picker shows. A read-only mount disables all three buttons and leaves the picker usable. Lamina saves these changes with your other unsaved changes when you click **Save all**.

The **Aggregation** picker is not the **Risk aggregation** setting further down the drawer. **Risk aggregation** chooses the profile that rates the Risks of the model; see [Risks](/docs/sightline/usage/lamina/risks).

## Exporting the diagram

The **PNG** and **Draw.io** exports capture the diagram. On a model with no data slices, Lamina disables them until you create one, though reports stay available.

The toolbar's Export menu can also write one PNG for each saved slice. It appears once the model has a saved slice, so a slice that is not yet saved does not count. See [PNG export](/docs/sightline/usage/lamina/png-export) for the file names, the framing and what the batch reads from disk.
