The Model Tree is a section in the Sightline activity-bar view, below Models. It lists the entities of the open model. It shows where an entity sits, finds an entity by name, and moves the canvas to it without panning.

## Opening the tree

1. Select **Sightline** in the activity bar.
2. Find the Model Tree section, below Models.

Select the section header to collapse or expand it. Drag the divider above the header to change the height.

The Journal section sits below the Model Tree. It starts collapsed and follows the focused model in the same way. See [Undo, redo and the journal](/docs/sightline/usage/shared/licensing#undo-redo-and-the-journal).

The section appears only while a model of that app is open in an editor.

The tree follows the editor you are looking at. Open two models and select between their tabs, and the tree shows the entities of whichever one has focus.

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

## Rows the canvas is not showing

With a data slice active, the canvas renders only the entities the slice keeps. The tree still lists the whole model, and it greys the rows that the slice leaves off the canvas. Hover a greyed row to see the reason in its tooltip.

A greyed entity still exists in the model. Double-click its row to open its props drawer and edit it, because editing reads the whole model rather than the slice. A single click highlights the row and leaves the canvas alone, because there is no node to move to.

## Section size and state

VS Code owns the section. It remembers whether you left it collapsed or expanded, and how tall it is, alongside every other view in your workspace. Use **Reset View Locations** from the Command Palette to return the whole sidebar to its defaults.

## Tetra

The Models section above the tree lists the workspace's `*.tetra.yaml` files. Select one to open it, and the tree below shows its entities.

The tree shows containment: zones, groups, devices, networks and information nodes, each under its parent. Channels, flows and annotations are relations rather than tree members.

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

A greyed row has the tooltip `not in the active data slice`. Both filter modes produce greyed rows:

- **Prune** removes the non-matching entities from the canvas, so every one of them greys in the tree.
- **Shade** keeps them on the canvas at 30% opacity, and greys the same rows in the tree.

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

The Models section above the tree lists the workspace's `*.bowtie.yaml` files. Select one to open it, and the tree below shows its entities.

The tree groups the model by entity type. Causes, events and outcomes each sit under a group header. Flows between causes, events and outcomes do not appear as rows.

A control belongs to the cause or the outcome that carries it. It renders under that line rather than under a header of its own. Expand a cause to see the controls on it.

Two groups collect the controls that cannot nest:

- **Shared controls.** More than one line references the control. Lamina lets you paste a control as a reference. Both lines then carry the same canonical id, so the tree cannot file it under either one.
- **Unattached controls.** No cause or outcome references the control. It is either an entry in the top-level `controls:` list that nothing uses, or a reference that does not resolve. Both are usually worth fixing in the model.

A shared control has a node on every line that carries it. Selecting its row centres the viewport on all of them together.

Lamina's tree selects one row at a time. Cmd, Ctrl and Shift clicks on its rows act as a plain click.

### On the Tabular Editor

The tree lists the whole model. No row greys, because the table shows every entity whatever data slice is active.

| Action | Result |
|---|---|
| Single click on a row | The table ticks that entity's row and scrolls it into view. |
| Tick rows in the table | The tree highlights the matching rows. |
| Double-click on a row | The props drawer opens for that entity. |

With a **Group** active, a click first opens the group that holds the row. A single click never opens the props drawer.

A single click on a tree row replaces the table selection with that one row. Ticking several rows in the table highlights every one of them in the tree.

A control has no row in the table, so its tree row highlights and the table ticks nothing. A Type or Filter condition can hide the row of a tree-selected entity. The tree row highlights, the table ticks nothing, and the condition stays as it is.

A selection made in the table does not select nodes on the diagram.

For how a slice decides which entities reach the canvas, see the [Lamina Data Slices](/docs/sightline/usage/lamina/data-slices) page. The Tetra tree uses the same control over a containment tree.
