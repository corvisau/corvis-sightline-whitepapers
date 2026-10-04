Use the stack picker to act on one object among several overlapping items on
the Tetra canvas, without zooming in or rerouting the diagram.

## Opening the stack picker

**Cmd/Ctrl+right-click** a point on the canvas where multiple nodes and/or
edges overlap. A list of every item under the cursor appears, ordered
front-to-back.

(Shift+right-click is a different gesture. It opens the nudge popover directly
for the item under the cursor. The Open Nudge Pane action in the Per-item
flyout actions section is the stack-picker equivalent.)

## Per-item flyout actions

Hover a row in the stack-picker list to open a flyout to its right. The flyout
holds two actions above that item's normal context menu, separated by a divider:

| Action | What it does | Availability |
|--------|--------------|---------------|
| **Select** | Selects that specific item, same as clicking the row. Works for both nodes and edges. | Always enabled. |
| **Open Nudge Pane** | Opens that item's nudge popover directly (the same panel Shift+right-click opens on the canvas): the +/- standoff controls for a channel/flow edge, or the position controls for an annotation. | Enabled for edges (channels and information flows) and annotation nodes. Disabled for devices, zones, groups, and networks - they have no nudge popover. |

Both actions close the stack picker after running.

Shift+right-click is not available inside the picker, because you hold
Cmd/Ctrl to keep the picker open. Open Nudge Pane takes its place.

## What the picker can see

The picker lists the objects under the cursor. A system container is click-through over its interior, so that you can click its own member devices (see [canvas stacking order](/docs/sightline/usage/tetra/canvas-z-order)). Without a special rule, the picker would leave the system out exactly where the picker is most useful. A system larger than the viewport would be reachable only by panning to find a border.

Tetra therefore adds a system to the list when the cursor lies inside its box. The test uses screen positions, so it stays correct at any zoom or pan. The system comes after the objects that the canvas reports, so the topmost object still leads the list. The list never shows an object twice.
