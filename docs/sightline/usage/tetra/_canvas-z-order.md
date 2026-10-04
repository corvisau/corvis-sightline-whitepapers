This page lists how the layers of the Tetra canvas stack, from bottom to top. It also says which layers take a click and which let it pass through.

## The stack, bottom to top

1. Zone fills
2. Group fills
3. Network bars
4. Device boxes
5. Information badges on devices and zones
6. System overlays
7. Overlay regions (the shaded regions authored in YAML; see [Overlays](/docs/sightline/usage/shared/overlays))
8. Channels and information flows
9. Edge label pills
10. Findings badges and coverage badges, on edges and on nodes
11. Annotation leaders (the lines and arrows)
12. Annotation bubbles

Channels and flows paint above every node fill.

A bubble paints above its own leader, so the leader tail tucks behind the bubble.

Where two badge layers cover the same pixel, the later layer in items 9 and 10 wins. The order is: edge label pills, edge findings badges, edge coverage badges, then node coverage badges.

### Order among channels and flows

Information flows paint above named channels. Named channels paint above shorthand channels (the automatic device-to-network connections).

An edge label pill does not hide behind a crossing edge, because pills paint in their own layer above all edges.

## Layers above the canvas

The legend overlay, context menus, drawers and dialogs sit above the whole canvas.

## Paint order and click order

A click normally goes to the layer painted on top. Two layers paint above objects that a click must reach, so they take clicks only where nothing else needs them.

| Layer | Paints above | Takes clicks on |
|---|---|---|
| System overlay | Its own member devices | A band of 8 px along its border only |
| Channels and flows | Every node fill | The transparent selection band of the edge, minus the run through any node that the route passes via |

**System overlay.** A system box is the union of its members plus 20 px of padding, so it covers every member. Only the border band takes clicks, and a click inside the box reaches the member below.

**Channels and flows.** A route that passes via a node is drawn straight through the node. That run takes no clicks inside the node. Selection comes from the transparent band alone, and every edge type has one.

A system is click-through over its interior, so the stack picker adds it back to its list by testing where the click landed. See the [Stack picker](/docs/sightline/usage/tetra/stack-picker) page.
