Layout and nudge tweaks are per data slice, not baked into the model. They cover edge routing (standoff, port, attach side, label
positions), container structure (layout direction, margins, min size,
alignment, annotation anchor, child order) and annotation-bubble placement. Each slice carries its own
overrides; a sub-slice inherits its base slice's layout and stores only
what it changes on top.

---

## Where overrides live

A `DataSlice`'s `viewOptions.overrides` holds four maps, each keyed
by entity id:

| Key | Entity | Overridable fields |
|---|---|---|
| `channels` | Channel | `startEdge`, `endEdge`, `startStandoff`, `endStandoff`, `startPort`, `endPort`, `labelPositions`, `viaOffset` (slice-only — see below), and `hops.<hopNodeId>.{startEdge, endEdge, startStandoff, endStandoff, startPort, endPort, viaOffset}` |
| `flows` | Information flow | same as `channels` |
| `containers` | Zone / group / device / network / information | `layout`, `marginTop`, `marginLeft`, `marginRight`, `minWidth`, `minHeight`, `matchSiblingSize`, `align`, `valign`, `annotationPosition`, `annotationPositionOffset`, `memberOrder` |
| `annotations` | Annotation link, by index within its `links[]` array | `positionNode`, `positionLine`, `positionX`, `positionY`, `maxWidth`, `distance` |

A slice with no tweaks for an entity or field omits that key. The
schema never writes an empty placeholder, so a slice's manifest YAML shows
only what it changed:

```yaml
dataSlices:
  - id: ot-view
    name: OT View
    viewOptions:
      overrides:
        channels:
          ch-plc-scada:
            startStandoff: 40
            hops:
              sw-core:
                startEdge: right
                endStandoff: 30
        containers:
          z-ot:
            layout: columns
            memberOrder: [net-dmz, d-web]
```

A path hop's own `id` and its `via` (which follows this channel's waypoints) stay model-side. So do the deprecated `portOffset` / annotation `position` / `linePosition` /
`positionOffset` aliases. None of these is part of this system. A hop's routing geometry and its `viaOffset` are part of it (see below).

### Per-hop routing overrides (`hops`)

A channel or flow that routes through intermediate nodes carries a `path[]`,
and each hop on it has its own attach edges and standoffs. Those are
overridable per slice under `hops`, keyed by the hop's node id, not by
its position in `path[]`:

- Ids, not indices, for the same reason `memberOrder` stores ids (below). An index would not survive the first structural edit, and inserting a hop
  is the ordinary way a path grows.
- A key naming a hop that is no longer in the path is ignored. The
  override degrades rather than mis-applying when the model moves on.
- A node that appears twice as a hop in one path shares a single entry. Both occurrences get the same routing.

`startPort` / `endPort` apply to a hop as well as the root. A hop's
`startPort` is the port the line arrives on and its `endPort` the port it
leaves from, matching the entry/exit naming below. An explicit hop port
overrides the router's own choice, including the perpendicular-transit
alignment. Leave it unset to keep the automatic behaviour.

**Per-hop geometry is slice-only.** The model's `path[]` entries carry routing
intent (the hop's `id` and its `via`) and nothing positional. `startEdge`,
`endEdge`, `startStandoff`, `endStandoff`, `startPort`, `endPort` and
`viaOffset` can be authored only on a slice.

**`viaOffset` is slice-only at both levels:** on a hop, and on the relationship
itself (the default every one of its `via` hops inherits). `via` names which
route the traffic borrows and stays model-side. `viaOffset` says where to
draw that borrowed route, which is presentation.

> ⚠️ Tetra ignores any of these fields written in the model YAML, without an
> error. Set them in the segment popover instead.

The model names these fields from the hop rather than the line. A hop's `startEdge` and `startStandoff` are its **entry** geometry
(the line arriving from the previous node). Its `endEdge` and `endStandoff` are its **exit** geometry
(the line leaving toward the next one).

> **Editing:** Shift+right-click the run of line you want on the canvas. The
> segment nudge popover is the only editing surface for routing. The props
> drawer's Advanced-layout fields, which wrote the model, are gone. The popover
> speaks in segments and undoes the naming inversion above. The page
> [Path segments and the segment nudge popover](/docs/sightline/usage/tetra/path-segments) explains which control lands in which
> field. It also covers the cases (a `via`-routed relationship, a collapsed hop) where
> a segment is reachable only through the popover's header stepper.

## Base-slice inheritance

Tetra data slices support a `baseSlice` chain for filters (see
[Tetra Data Slice Filtering](/docs/sightline/usage/tetra/data-slice-filtering)). A slice can name
another slice as its base, and the base's filter applies first. Layout
overrides reuse the same chain for inheritance:

- Walking a slice's `baseSlice` outward, the nearest ancestor that
  defines a given `(entity, field)` wins.
- Resolution is field-level. A sub-slice can override
  just `startStandoff` on a channel. Every other field still inherits from the base: the channel's `endStandoff` and all other channels, containers and annotations.
- A cycle or a dangling `baseSlice` reference degrades to no inheritance
  for that slice, as in the filter chain.

So a base slice such as OT View can lay out about 80% of a diagram. A sub-slice
such as OT View: Site 42 layers on just the two or three nudges specific to
that view.

## Editing overrides

**Edge routing** has its own shift+right-click nudge panel (Position: Start/
End offset, port, attach edge, label positions).

**Container structural fields** have their own Shift+right-click **Container Properties** panel on a zone, group, device, network or information node. These fields are `layout` direction, `minWidth` and `minHeight`, margins, `align` and `valign`, `matchSiblingSize`, and (zone and group only) `annotationPosition` and `annotationPositionOffset`. This panel is the only place to edit them. The props
drawer's Advanced section does not show them.

**Annotation placement** has its own shift+right-click nudge panel (Side /
Line / X / Y), shared with Lamina.

All three panels edit the active slice's `overrides`. Nothing round-trips
through the model file, whichever slice is active (see the section "None — full
model" below). The canvas updates immediately, and the change is written on
**Save** with the rest of the unsaved changes.

### Child order (`memberOrder`)

Child order is edited by dragging rows in the props drawer's Children
list on a zone or group. There is no nudge panel for it. The drag writes
the active slice's `memberOrder` and never reorders the model's
`members` array. Like every other drawer edit, it buffers in the pool of unsaved changes
and commits on **Save**. A reorder, an add and a removal made together
save (or discard) together.

`memberOrder` is a sparse, ordered list of child ids, not indices:

- Ids in the list lead, in the order given.
- Ids not mentioned keep their natural relative position, after the listed ones.
- Ids naming a child that no longer exists are ignored.

So the override survives the model changing underneath it: adding, removing
or renaming a child degrades the ordering gracefully instead of scrambling
the container. Only children that are not `information` participate. A zone's
information badges are laid out on their own path beneath the container and
hold their positions regardless.

Ordering changes the layout structure: the layout groups consecutive
same-category children into segments, and networks and containers each force
a segment break. Moving a device from one side of a network to the other
therefore changes the container's row and column structure.

To reorder the model's authored `members` array itself, edit the YAML.

### "None — full model"

Selecting **None** in the Data slice picker views the raw, unsliced model.
While **None** is active, nudging does not edit the model. Every field in the three nudge panels renders
disabled with a "Select a data slice to edit" hint. The Children
list's drag handles are disabled under the same rule, with the same hint.
Select a real slice to make any layout or nudge field editable.

### Inherited vs. local (nudge panel hints)

When a slice is active, all three nudge panels and the Children list (for
`memberOrder`) show, per field:

- **`inherited from <base slice name>`**: the value comes from a base
  slice, and the active slice does not touch the field.
- **`Reset to inherited`**: the field is locally overridden on the active
  slice. Clicking it clears the local override, so the field falls back to whatever the
  base chain (or nothing) provides.
- Neither: the field has no override anywhere in the chain, so the model value is in effect.

When no data slice is active (**None**), every field is disabled instead.
A single panel-wide "Select a data slice to edit" hint replaces the
per-field ones.

The Children list carries one hint for the whole list, because `memberOrder` is a single field describing the entire order. The hint reads `order inherited from <base>` when the order comes from an ancestor. A `Reset order to inherited` button appears when this slice sets it.

### Canvas-wide indicator

The nudge-panel hints above answer the question one field at a time. To see
the shape of a slice's whole deviation from its base at a glance, turn on
**Layout Overrides** in the View menu. With it on, every container, channel,
flow and annotation that carries a layout override in the active slice shows a
small dot:

- **Filled**: the override is set locally on the active slice.
- **Hollow (a ring)**: the override is only inherited from a base slice, and
  the active slice does not touch it.

The dot is entity-level: it marks whether an entity carries an override
and whether any of it is local, not which specific field. An entity with
both a local field and an inherited one shows filled, since a local
override is present. For per-field or per-hop detail, use the nudge-panel
hints above or the override inspector below.

The toggle has no effect when no data slice is active (**None**).

### The override inspector

The hint above shows whether a value is local. The inspector shows which slice set it and what that value overrides.

Open it with **Inspect overrides** in the segment nudge popover. It lists every
layout override the channel or flow carries, and under each one every slice in
the chain that sets it:

- The slice in force is first, marked **set here** when it is the active slice.
- Every slice it supersedes is listed below it, struck through. Each shows the
  value that would apply if you cleared the winner.
- A slice that sets nothing for that field does not appear.
- Per-hop overrides are grouped under the hop they belong to. They are labelled
  **entry** and **exit** rather than start and end, because a hop's `start*`
  fields describe the line arriving at it.
- An override keyed to a hop the path no longer routes through is not shown.
  Overrides are keyed by hop node id, so one survives a path edit that removed
  its hop. The inspector ignores it, as the renderer does.

Click any row to switch the active slice to the one that set that value. The
popover and the dialog both close, and you land in the slice where changing or
clearing that value takes effect.

The inspector is read-only. It shows where a value lives and takes you there.
Change a value in the nudge panel's own controls, or use **Reset to inherited** for the active slice's own
override.

With no data slice active there is no chain to inspect, so no Inspect button
renders.

## Layout in model YAML

Layout fields authored in `channels:`, `flows:` and container YAML still render.
That layout is not slice-scoped.
