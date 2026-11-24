The record drawer shows one record of a rule in full. Tetra and Metron open it from the Records tab. The drawer is read-only: it changes no record.

## Open a Record

1. Open the **Records** tab.
2. Double-click a row, or select a cell and press Enter.

A single click selects the row and opens nothing. Press Escape or select **Close** to close the drawer.

The drawer opens wide, at 720px. Drag its left edge to resize it, and double-click the edge to return to 720px. Every wide drawer remembers one width.

## Record Drawer Sections

| Section | Content |
|---|---|
| Header | The title of the record and its state: Current, Changed or Closed. Below it, the rule, the package, the model and the status of the record. |
| Key | One row for each key part of the record. A part that names an entity of the model shows as a link. |
| Data | The label of the record, its tags as pills, and the lists `zones`, `channels`, `flows` and `pivots`. An id in a list that names an entity of the model shows as a link. Below them, the whole data of the record as a tree. |
| Journal | The journal of the record, oldest entry first: the time, the kind of entry and who made it. |
| Linked by | The number of Lamina causes that link the record, and the name of each cause. |

The Journal and **Linked by** sections show the entries, the count and the causes that the tab loaded for the selected record. The drawer shows no journal entries for a record that the tab has not finished loading.

A data field named `label`, `tags`, `zones`, `channels`, `flows` or `pivots` shows in the list above the tree only when its value has the expected type: text for `label`, and a list of text values for the others. Any other value shows in the tree only.

The **summary** column of the Records tab shows the label of the record. A record with no label shows its first three text, number or boolean fields as `name: value`.

## Read the Data Tree

The tree shows each field of the data with its value. Text shows in quotes and in the link colour, numbers and booleans in bold, and `null` in italics. A list shows its length in square brackets, and an object shows its field count in braces. A text value over 200 characters shows its first 200 characters.

- Select the arrow, or press Enter or Space on a row, to expand or collapse it. The right arrow expands a row and the left arrow collapses it.
- Select the first copy icon of a row to copy its path, for example `items[1].name`.
- Select the second copy icon to copy its value as JSON. The copy takes the whole text value.

## Open a Record from Test Rule

**Test rule** in the rule drawer lists the records that a test run would write, grouped as created, changed, closed and returned, with a count for each kind. Select **Open** on a row to see that record in the record drawer. The record drawer opens beside the rule drawer, so both stay in view.

A test run writes nothing, so the drawer shows only what the run would write. It shows the key parts and data from the run, one journal line for the run, and no **Linked by** section. Press Escape or select **Close** to close the record drawer and keep the rule drawer open. Pressing **Test rule** again closes it.

## Open a Record from a Cause in Lamina

In the props drawer of a cause, select the name of a record in the **Records** section to open it in the record drawer. The record drawer opens beside the props drawer, so both stay in view. Press Escape or select **Close** to close the record drawer and keep the props drawer open.

When a link has changed since its record was accepted, the drawer adds a **Change since accepted** section. It lists each changed field with its accepted value and its current value side by side. A link that is not saved yet opens with the name that the link holds and no changed fields. This drawer shows no journal entries and no **Linked by** section.

## Open an Entity, a Cause or the Model

In Tetra and in Metron, the drawer connects to the Tetra model that the window shows.

- An id that names an entity of the model shows the label of the entity as a link. Select the link to open the model in its diagram with that entity selected.
- An id that the model does not hold, or that more than one entity holds, shows as plain text.
- Select the name of a cause in **Linked by** to open the Lamina model with that cause selected. A cause with no label shows by its id.
- Select **Open in Tetra** to open the model in its diagram. The diagram selects the first entity that the record names and the model holds. The key parts come first, then the lists in the order above. A record that names no such entity opens the model with nothing selected.

When the window cannot read the model, every id shows as plain text and **Open in Tetra** opens the model with nothing selected.

## Footer

**Copy id** copies the id of the record. **Open in Tetra** shows in the drawer of the Records tab, in Tetra and in Metron.
