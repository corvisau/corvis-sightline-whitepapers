An overlay is a render-only shaded region drawn on top of the diagram around a set of entities. Overlays are non-interactive: they cannot be clicked or dragged.

Neither Tetra nor Lamina has a dialog or panel to create or edit overlays yet. A model that already holds overlays shows them on the canvas.

## What an overlay shows

- A region that encloses its member entities, shaded in the overlay colour.
- An optional label in the top-left corner of the region, in the overlay colour.
- An outline that follows the members, either as one bounding rectangle or as a tight outline around each member.

A member that the current diagram does not contain is skipped. An overlay with no member in the diagram is not drawn.

## Tetra

A Tetra overlay can enclose devices, zones, groups and networks. Tetra draws overlays above the zone, group, network and device fills, and below edges and annotations. See [Tetra canvas stacking order](/docs/sightline/usage/tetra/canvas-z-order) for the full order.

## Lamina

A Lamina overlay can enclose causes, events, outcomes and controls. Lamina draws overlays above the node boxes and below annotations.
