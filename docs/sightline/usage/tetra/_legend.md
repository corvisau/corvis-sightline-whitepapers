How to show, move, and export the diagram legend on the Tetra canvas.

## Showing / hiding the legend

Open the **View** popover in the toolbar. Set **Legend** to **Show**. The row sits
with the other view-option rows, such as Waypoints and IDs.
Setting it back to **Hide** removes the legend again. The legend renders only
when the current diagram has channels, information, line styles, directions, zones, or
groups to show. An empty diagram shows nothing even with **Legend** set to **Show**.

## What the legend shows

The legend lists only what the diagram uses. A section with nothing to list does
not appear.

- **Channels:** one row per channel type, with its colour and line style.
- **Information:** one row per information type: System, Function or Security.
  The colour is the one the information badges and the flow lines use. A type
  appears when a flow or a badge on the canvas has it. A badge with no
  information type has no row.
- **Line style:** one row per way the flows are drawn. A flow is solid when it is
  Programmatic, dashed when it is Interactive, and dotted when it is Offline.
  The section appears when the diagram has flows.
- **Directions:** one row per flow or channel direction in use.
- **Zones:** one row per zone type, with its colour.
- **Groups:** one row per group style in use.

## Moving the legend to a corner

Right-click the legend to open the **Legend position** menu. For keyboard
access, press Tab to focus the legend, then press the Menu key or Shift+F10.
The menu lists all four canvas corners: Top-left, Top-right, Bottom-left and
Bottom-right. The current corner is marked. Pick a different corner to move
the legend there. The choice persists with the rest of the view options. **Save** in the
toolbar becomes enabled once you change it.

## Size

The legend renders at a larger, more readable size than the rest of the
canvas UI. The exported legend PNG (see below) uses the same size.

## Exporting the legend as a PNG

Use **Export Legend PNG** in the toolbar's export menu. It saves the legend as
a standalone image, whether or not the canvas currently shows it. Use the
image to place the legend in a report or slide without the full diagram
behind it.
