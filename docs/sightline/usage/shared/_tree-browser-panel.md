The Model Tree is a section in the Sightline activity-bar view, below Models. It lists the entities of the open model. It shows where an entity sits, finds an entity by name, and moves the canvas to it without panning.

## Opening the Tree

1. Select **Sightline** in the activity bar.
2. Find the Model Tree section, below Models.

Select the section header to collapse or expand it. Drag the divider above the header to change the height.

The Journal section sits below the Model Tree. It starts collapsed and follows the focused model in the same way. See [Undo, redo and the journal](/docs/sightline/usage/shared/licensing#undo-redo-and-the-journal).

The section appears only while a model of that app is open in an editor.

The tree follows the editor you are looking at. Open two models and select between their tabs, and the tree shows the entities of whichever one has focus.

When a Lamina model and a Tetra model are both open, the activity bar shows the sections of one app only: the app whose model tab you selected last. That covers the Model Tree and, for Lamina, Elements, or for Tetra, Slices. Select a tab of the other app, and its sections replace them. Select a tab that is not a Lamina or Tetra model, such as a README, and the sections stay. Close every tab of the app that is showing, and the sections of the other app appear.

On the Tabular Editor the tree stays available in both apps. It works as described under [Tetra](#tetra) and [Lamina](#lamina).

## Filtering

Type in the filter box to narrow the tree by label. The tree keeps each matching row and the rows above it, and it opens the branches that hold a match. Clear the box to restore your own expansion state.

## Selecting

A single click selects. A double-click edits. A single click never opens the props drawer.

| Action | Result |
|---|---|
| Single click on a row | The tree highlights the row. The canvas selects the matching node and centres the viewport on it. |
| Single click on a canvas node | The tree opens the branches above that entity, scrolls its row into view. |
| Double-click on a row | The props drawer opens for that entity. |

## Rows the Canvas Is Not Showing

In Tetra, with a data slice active, the canvas renders only the entities the slice keeps. In Browse the tree lists only those entities. Otherwise the tree still lists the whole model, and it greys the rows that the slice leaves off the canvas. Hover a greyed row to see the reason in its tooltip.

A greyed entity still exists in the model. Double-click its row to open its props drawer and edit it, because editing reads the whole model rather than the slice. A single click highlights the row and leaves the canvas alone, because there is no node to move to.

## Section Size and State

VS Code owns the section. It remembers whether you left it collapsed or expanded, and how tall it is, alongside every other view in your workspace. Use **Reset View Locations** from the Command Palette to return the whole sidebar to its defaults.

## Tetra

In the browser, select **Tools**, then **Model tree** in the Tetra toolbar. The tree opens beside that model's editor and provides the same selection, filtering and editing actions. Close it with the panel's close button. The browser panel has a fixed width; the extension uses the size of the VS Code sidebar section.

In Tetra, the tree has two modes. See [Build and Browse](#build-and-browse). In Build, every row except a type root carries a toggle that includes or excludes the row in the active data slice. A folder row carries one too. The tree greys nothing out. A right-click on an entity row opens the same menu as a right-click on that entity in the diagram. Below it, the menu lists **Hide others** and **Revert all**. See [Choosing what a slice shows from the Model Tree](/docs/sightline/usage/tetra/data-slice-filtering#choosing-what-a-slice-shows-from-the-model-tree).

The entity menu has the same entries, in the same order, as the diagram menu for that type of entity. **Layout** opens a submenu with **Rows** and **Columns**. Right-click a row that is not selected, and the menu applies to that row. Right-click a selected row when several rows are selected, and the menu applies to the whole selection, as in the diagram. In Browse, every entry that changes the model is disabled. **Edit…** and **View Code** stay available. A type root, a folder and an orphan bucket have no entity menu.

The Models section above the tree lists the Tetra models in the workspace. Select one to open it, and the tree below shows its entities.

Each row shows a glyph for its type, such as a zone, a group, a device, a network, an information node, a channel, a flow, a system or an annotation. The row carries no type word. Hover a row to see its kind in the tooltip.

The tree shows containment first: zones, groups, devices, networks and information nodes, each under its parent. Channels, flows, systems and annotations follow, each under a root of its own named **Channels**, **Flows**, **Systems** or **Annotations**. A root appears when the model holds at least one entity of that type, and it opens closed. A root has no toggle and no props drawer. Select an entity under it, double-click the entity to open its props drawer, or find it with the filter. Overlays do not appear in the tree.

### Build and Browse

The Tetra Model Tree has two modes. The **Build** and **Browse** buttons at the left of the tree toolbar switch between them. A model opens in Browse, and the tree returns to Browse each time you open the model.

| | Browse | Build |
|---|---|---|
| Rows listed | The entities the active data slice shows, and the rows above them | Every row of the model |
| Row toggles | None | One on each row except a type root |
| Folder actions and drag to move | Not offered | **New folder**, **Rename**, **Delete folder** and drag |
| **Hide others** and **Revert all** | Not offered | In the row menu |
| Entity menu | Every entry that changes the model is disabled | As the diagram menu, with the changing entries enabled |

Browse changes nothing, so you can move around a model without touching the data slice. A type root or a folder appears in Browse only while it holds a row that the slice shows. A row that the slice leaves out is not listed, and nothing greys out.

A click, a double-click and the filter box work in both modes. A click on a row selects it and centres the canvas on it, and a double-click opens the props drawer. Switching mode returns the tree to its starting depth.

A model that you cannot edit always shows Browse, and its toolbar has no mode buttons. On the Tabular Editor the table lists the whole model, so Browse lists every row there. The Lamina Model Tree, the Elements view and the Slices view have no modes.

### Folders for Flows, Channels and Annotations

A flow, a channel or an annotation can sit in a folder. A folder appears as a row under the root of its type, and folders nest. Folders organise the rows of the tree. They are not part of the model's containment, and a folder has no props drawer. A folder appears when an entity sits in it or when the model lists it, so an empty folder stays in the tree. Systems have no folders.

| Action | Result |
|---|---|
| **New folder** on a root or a folder row | Adds a folder under that row and opens its name field. |
| **Rename** on a folder row | Renames the folder. The rows inside it, and the folders beneath it, follow. |
| **Delete folder** on a folder row | Removes the folder. A folder that holds a row or another folder cannot be deleted. |
| Drag rows onto a folder row | Moves the rows into that folder. |
| Drag rows onto the empty area under the tree | Moves the rows to the top level of their own root. |

A drag moves rows of one type at a time. A row cannot move into a folder of another type, and the tree says so in a notice. Moving a folder moves everything inside it.

Each change waits as an unsaved change until you click **Save all**. A folder name cannot contain `/`, and two folders under one parent cannot share a name. A reader who cannot edit the model sees Browse, which offers none of these actions.

A folder row's toggle sets the state of every entity inside it, at every depth. The row shows a state only when all of those entities share it. The row menu offers **Revert all**, which clears the states of the entities inside the folder, and does not offer **Hide others**.

Every root opens on first paint. Deeper branches stay closed until you click their arrow.

The filter match ignores case.

Switching between the diagram and the Tabular Editor keeps your open branches and filter text.

Five gestures differ from Lamina:

- Clicking a canvas node opens the ancestors of that entity, highlights its row, and leaves the props drawer closed.
- Hold Cmd (macOS) or Ctrl (Windows, Linux) and click more canvas nodes to select several. The tree highlights the row of every selected node, and the canvas and the tree stay in step as you add or remove nodes.
- Hold Cmd (macOS) or Ctrl (Windows, Linux) and click a row to add it to the selection, or to remove it again. The canvas selects every node in the set.
- Hold Shift and click a row to select every visible row between the last row you clicked and this one. Rows inside a collapsed branch and rows the filter hides are not part of the range, and group headers are skipped.
- Clicking empty canvas, or pressing Escape, clears the whole selection and the tree highlight, and closes the props drawer. While a dialog, a popover or a menu is open, Escape closes it and the selection stays. Press Escape again to clear the selection. Escape leaves the selection alone while you type in a field.

The canvas cannot always move to the entity you clicked. A node inside a collapsed zone, or one that a View menu toggle hides, has no box to centre on. The viewport then stays where it is, and the row still highlights.

A greyed row can join a multi-selection. Its row highlights, and the canvas skips it because there is no node to select.

A greyed row has the tooltip `not in the active data slice`. The slice removes the non-matching entities from the canvas, so every one of them greys in the tree.

The View menu's "Hide ..." toggles declutter the canvas without greying anything in the tree.

For how a data slice narrows the canvas, see the [Tetra Data Slice Filtering](/docs/sightline/usage/tetra/data-slice-filtering) page.

### On the Tabular Editor

The tree lists the whole model. No row greys, because the table shows every entity whatever data slice is active.

| Action | Result |
|---|---|
| Single click on a row | The table ticks that entity's row and scrolls it into view. |
| Cmd or Ctrl click, or Shift click, on rows | The table ticks every row in the resulting set. |
| Tick rows in the table | The tree highlights the matching rows. |
| Double-click on a row | The props drawer opens for that entity. |

With **Tree** on, a click first opens the rows above the entity. With a **Group** active, it opens the group that holds the row. A single click never opens the props drawer.

Channels, flows, systems and annotations are not tree members. Ticking one highlights no tree row, and a tree click never ticks one. A channel you ticked in the table stays ticked.

A Type or Filter condition can hide the row of a tree-selected entity. The tree row highlights, the table ticks nothing, and the condition stays as it is. When **Tree** is on and the condition removed only an ancestor, the table shows that ancestor greyed and scrolls to it without ticking it.

A selection made in the table does not select nodes on the diagram.

## Lamina

The Models section above the tree lists the Lamina models in the workspace. Select one to open it, and the tree below shows its entities.

The tree groups the model by entity type. Causes, events and outcomes each sit under a group header. Flows between causes, events and outcomes do not appear as rows.

A control belongs to the cause or the outcome that carries it. It renders under that line rather than under a header of its own. Expand a cause to see the controls on it.

Two groups collect the controls that cannot nest:

- **Shared controls.** More than one line references the control. Lamina lets you paste a control as a reference. Both lines then carry the same canonical id, so the tree cannot file it under either one.
- **Unattached controls.** No cause or outcome uses the control, or a reference to it does not resolve. Both are usually worth fixing in the model.

A shared control has a node on every line that carries it. Selecting its row centres the viewport on all of them together.

On the **Table** surface, Lamina's tree selects one row at a time, and Cmd, Ctrl and Shift clicks act as a plain click. On the **Diagram** surface, the tree lists the Risks and carries buttons for them, and those clicks highlight several rows. See [Model Tree buttons](/docs/sightline/usage/lamina/risks#model-tree-buttons).

### On the Tabular Editor

The tree lists the whole model. No row greys, because the table shows every entity whichever Risk the canvas shows.

| Action | Result |
|---|---|
| Single click on a row | The table ticks that entity's row and scrolls it into view. |
| Tick rows in the table | The tree highlights the matching rows. |
| Double-click on a row | The props drawer opens for that entity. |

With a **Group** active, a click first opens the group that holds the row. A single click never opens the props drawer.

A single click on a tree row replaces the table selection with that one row. Ticking several rows in the table highlights every one of them in the tree.

A control has no row in the table, so its tree row highlights and the table ticks nothing. A Type or Filter condition can hide the row of a tree-selected entity. The tree row highlights, the table ticks nothing, and the condition stays as it is.

A selection made in the table does not select nodes on the diagram.

For how the picked Risk decides which entities reach the Lamina canvas, see [Risks](/docs/sightline/usage/lamina/risks). The Tetra tree uses the same control over a containment tree.
