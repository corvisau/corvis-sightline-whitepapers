How to show, move, and export the diagram legend on the Tetra canvas.

## Showing / Hiding the Legend

Open the **View** popover in the toolbar. Set **Legend** to **Show**. The row sits
with the other view-option rows, such as Waypoints and IDs.
Setting it back to **Hide** removes the legend again. The legend renders only
when the current diagram has channels, information, line styles, directions, zones, or
groups to show. An empty diagram shows nothing even with **Legend** set to **Show**.

## Legend Contents

The legend lists only what the diagram uses. A section with nothing to list does
not appear.

- **Channels:** one row per channel type in use (Network, Virtual or Hosting), with its colour and line style. The Hosting sample line ends in the same two markers the canvas draws: a filled diamond at the host and a hollow ring at the guest. A channel with its own line style or colour still shows the row of its type.
- **Information:** one row per information type: System, Function or Security.
  The colour is the one the information badges and the flow lines use. A type
  appears when a flow or a badge on the canvas has it. A badge with no
  information type has no row.
- **Line style:** one row per way the flows are drawn. A flow is solid when it is
  Programmatic, and dashed when it is Interactive.
  The section appears when the diagram has flows.
- **Directions:** one row per flow or channel direction in use.
- **Initiating end:** one row that shows the open circle on a flow line. The row
  appears when a flow in the model states an initiator.
- **Zones:** one row per zone type, with its colour.
- **Groups:** one row per group style in use.

## Moving the Legend to a Corner

Right-click the legend to open the **Legend position** menu. For keyboard
access, press Tab to focus the legend, then press the Menu key or Shift+F10.
The menu lists all four canvas corners: Top-left, Top-right, Bottom-left and
Bottom-right. The current corner is marked. Pick a different corner to move
the legend there. The choice persists with the rest of the view options. **Save** in the
toolbar becomes enabled once you change it.

## Size

The legend renders at a larger, more readable size than the rest of the
canvas UI. The exported legend PNG (see below) uses the same size.

## Exporting the Legend as a PNG

Use **Export Legend PNG** in the toolbar's export menu. It saves the legend as
a standalone image, whether or not the canvas currently shows it. Use the
image to place the legend in a report or slide without the full diagram
behind it.
