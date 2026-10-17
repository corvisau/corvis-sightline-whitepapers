Use **Move…** to place one node, or several nodes at once, under a different
parent on the Tetra canvas.

## Moving one node

Right-click a zone, group, device or network and choose **Move…**. The
picker dialog shows the zones and groups that can hold it.
Select a parent and choose **Move here**. The node keeps its place in the
canvas until you save, and **Undo** returns it to its previous parent.

## Moving several nodes

1. Hold **Cmd** (macOS) or **Ctrl** (Windows and Linux) and click each node to select it.
2. Right-click one of the selected nodes.
3. Choose **Move…**.
4. Select the new parent in the picker dialog, and choose **Move here**.

Every selected node moves to the parent you picked, in the order you selected
them. The nodes do not need to share a parent. **Undo** returns all of them in
one step.

## When Move… is not available

The menu omits **Move…** for a selection that contains:

- An annotation, an information item or a system.
- A node together with one of its own descendants. Select either the container or its contents.

The dialog dims every selected node and everything inside the selected nodes,
because a node cannot move into itself. When the selected nodes already share
one parent, the dialog starts on that parent. **Move here** stays disabled
until you pick a parent that is not dimmed and differs from the current one.

On a view where editing is not allowed, **Move…** appears disabled.

The Tabular Editor does not offer **Move…**. To change a value on several rows
there, use [bulk edit](/docs/sightline/usage/shared/bulk-edit).
