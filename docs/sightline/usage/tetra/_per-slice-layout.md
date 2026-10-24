Layout and nudge tweaks are per data slice, not baked into the model. They cover edge routing (standoff, port, attach side, label
positions), container structure (layout direction, margins, min size,
alignment, annotation anchor, child order) and annotation-bubble placement. Each slice carries its own
overrides; a sub-slice inherits its base slice's layout and stores only
what it changes on top.

---

## What a slice overrides

A slice holds overrides for three kinds of thing:

| Kind | What a slice can change |
|---|---|
| Channels and flows | Attach edge, offset and port at each end, label positions, the via offset, and the groups the line avoids |
| Containers | Layout direction, margins, minimum size, alignment, match sibling size, annotation anchor and child order |
| Annotations | Side, line position, X and Y position, maximum width and distance |

A slice with no tweak for an entity or field stores nothing for it.

### Per-hop routing

A channel or flow that routes through intermediate nodes has its own attach edges,
offsets and ports at each hop. A slice stores them per hop, so the segment popover
can edit any run of the line:

- A hop keeps its overrides by the hop's node, not by its position in the path.
  Inserting a hop therefore moves no override.
- An override for a hop that is no longer in the path is ignored. It applies again
  if the hop returns.
- A node that appears twice as a hop in one path shares one set of overrides. Both
  occurrences get the same routing.

An explicit port on a hop overrides the router's own choice, including the
perpendicular-transit alignment. Leave the port at its inherited value to keep the
automatic behaviour.

A hop's routing geometry and the via offset exist only in a slice. The model holds
which route a hop borrows, and a slice holds where to draw it.

The **Start** and **End** sections of the segment popover speak in segments. The
model names these values from the hop instead. A hop's entry is the line arriving
from the previous node, and its exit is the line leaving toward the next one.

> **Editing:** Alt+right-click the run of line you want on the canvas. The
> segment nudge popover is the only editing surface for routing. The page
> [Path segments and the segment nudge popover](/docs/sightline/usage/tetra/path-segments) explains which
> control changes which value. It also covers the cases (a via-routed
> relationship, a collapsed hop) where a segment is reachable only through the
> popover's header stepper.

## Base-slice inheritance

Tetra data slices support a base slice for filters (see
[Tetra Data Slice Filtering](/docs/sightline/usage/tetra/data-slice-filtering)). A slice can name
another slice as its base, and the base's filter applies first. Layout
overrides reuse the same chain for inheritance:

- Walking a slice's base chain outward, the nearest ancestor that
  defines a given value for a given entity wins.
- Resolution is field-level. A sub-slice can override
  just the start offset on a channel. Every other value still inherits from the base: the channel's end offset and all other channels, containers and annotations.
- A cycle or a missing base slice degrades to no inheritance
  for that slice, as in the filter chain.

So a base slice such as OT View can lay out about 80% of a diagram. A sub-slice
such as OT View: Site 42 layers on just the two or three nudges specific to
that view.

## Editing overrides

**Edge routing** has its own Alt+right-click nudge panel (Position: Start/
End offset, port, attach edge, label positions, and the groups to avoid).

**Container structural fields** have two surfaces. Alt+right-click a zone, group, device, network or information node to open the **Container Properties** panel. It sets minimum width and height, margins, alignment, match sibling size, and (zone and group only) the annotation position and its offset. Right-click the container and choose **Layout**, then **Rows** or **Columns**, to set the layout direction. The props
drawer's Advanced section does not show these fields.

**Annotation placement** has its own Alt+right-click nudge panel (Side /
Line / X / Y), shared with Lamina.

Drag a panel by its title bar to move it off the diagram. See [Moving a nudge panel](/docs/sightline/usage/shared/annotations#moving-a-nudge-panel).

All of these edit the active slice's overrides. Nothing round-trips
through the model, whichever slice is active (see the section "A model
with no data slices" below). The canvas updates immediately, and the change is
written on **Save** with the rest of the unsaved changes.

### Child order

Child order is edited by dragging rows in the props drawer's Children
list on a zone or group. There is no nudge panel for it. The drag changes the
active slice's child order and never reorders the model's
members; [File order](#file-order) does that. Like every other drawer edit, it buffers in the pool of unsaved changes
and commits on **Save**. A reorder, an add and a removal made together
save (or discard) together.

The order is a sparse list of children:

- Children in the list lead, in the order given.
- Children not in the list keep their natural relative position, after the listed ones.
- A child that no longer exists is ignored.

Adding, removing or renaming a child changes nothing else in the stored
order. Only children that are not `information` participate. A zone's
information badges are laid out on their own path beneath the container and
hold their positions regardless. [Information order](#information-order) covers
how to order them.

Ordering changes the layout structure: the layout groups consecutive
same-category children into segments, and networks and containers each force
a segment break. Moving a device from one side of a network to the other
therefore changes the container's row and column structure.

### File order

The **File order** list changes the order of a container's `members` in the model file. It is separate from the Children list: Children orders the active data slice's view, and File order orders the file for every slice.

**File order** is in the Advanced section of the zone, group and Model Properties drawers. It lists the container's authored members in the order the file gives them. To reorder, drag a row, or press Space on its handle, use the arrow keys, and press Space again to drop it. The list shows the new order at once. **Save** writes it with the rest of the unsaved changes, and only the `members:` order of that container changes in the file.

- A member that a filter places, written as `id` with `filter`, moves with its filter.
- A member that an `expr` entry adds is not listed. It has no position of its own, and Tetra writes the `expr` entries after the listed members.
- The list adds and removes nothing. Use the Children list for that.
- **File order** does not need a data slice. A model without a root container shows no list in Model Properties.

### Avoiding groups

A channel or flow can name groups that its route must go around. By default the
router avoids devices, networks and zones only, so a line can cut straight through a
group. An avoid list keeps one line out of the groups you choose.

> **Editing:** Alt+right-click the line to open the segment nudge popover. The
> **Avoid** section lists the visible groups with a checkbox each. Tick a group to
> route the line around it. Clear the tick to let the line cross it again.

The route keeps a clear gap of about 28 px from an avoided group, wider than the gap
it keeps from devices, networks and zones, so the detour is easy to see.

The list belongs to the whole channel or flow, so it applies to every segment of
the line, whichever segment the popover shows. A nearer slice replaces the list of its
base slice instead of adding to it. **Reset to inherited** removes the local list.

The router skips an entry that cannot apply, and the popover marks that entry with
the reason:

| Mark | Meaning |
|---|---|
| `holds an endpoint` | The group contains one end of the line, or its gap reaches one end. Routing around it would wall the line in. |
| `hidden group` | The group is hidden, so it is not an obstacle. |
| `not found` | The model has no group with that id. |

A skipped entry stays in the slice. It takes effect again if the model changes so
that it applies, and you can clear its tick at any time. Synthetic channels, which come
from a device's `connections` list, cannot hold an avoid list. Promote the channel first.

### Information order

The **Information** section in the props drawer of a device or zone sets the order of that owner's information badges. The canvas draws the badges in the order the list shows. The order belongs to the model, so every data slice shows the same order.

To reorder the badges:

1. Open the props drawer of the device or zone.
2. In the **Information** section, drag a row by its handle. Alternatively, press Space on the handle, use the arrow keys, and press Space again to drop it.
3. Select **Save**.

The new order stays with the other unsaved changes until **Save** writes it. Only the `members:` order of that device or zone changes in the file.

- The list orders the badges of one owner. It does not move a badge to a different device or zone.
- A read-only view disables the handles.
- The section also adds, relabels, retypes and deletes badges, on a zone as well as on a device.

### A model with no data slices

Layout overrides belong to a data slice, so a model with no slices has nowhere
to keep them. Its Diagram view shows an empty state instead of the canvas. A
props drawer opened from the Table view disables the Children list's drag
handles, with a "Create a data slice to edit the layout" hint.

### Inherited vs. local (nudge panel hints)

When a slice is active, all three nudge panels and the Children list show, per field:

- **`inherited from <base slice name>`**: the value comes from a base
  slice, and the active slice does not touch the field.
- **`Reset to inherited`**: the field is locally overridden on the active
  slice. Clicking it clears the local override, so the field falls back to whatever the
  base chain (or nothing) provides.
- Neither: the field has no override anywhere in the chain, so the model value is in effect.

The Children list carries one hint for the whole list, because the child order is a single value describing the entire order. The hint reads `order inherited from <base>` when the order comes from an ancestor. A `Reset order to inherited` button appears when this slice sets it.

### Pushing nudges to the base slice

The command **Tetra: Push Layout Nudges to Base Slice** moves the layout nudges of the active data slice into its base slice. It runs from the command palette and needs an open diagram.

In the browser, select **Tools**, then **Push layout nudges to base slice**. It opens the same confirmation and stages the same change.

1. Select the data slice whose nudges you want to move.
2. Open the command palette and run **Tetra: Push Layout Nudges to Base Slice**.
3. Read the confirmation. It names the slice and its base slice, and counts the elements that move and the base values that change.
4. Select **Push**.
5. Select **Save all** to write the change.

The base slice takes every nudge of the slice: channels, flows, containers and annotations. Where both slices set the same value, the base slice takes the value from the slice. The base slice keeps every value that the slice does not set.

The slice then holds no nudges of its own and inherits them from the base slice, so it draws the same layout as before. Other slices that share the base slice start to inherit the pushed nudges as well.

The change joins the other unsaved edits until **Save all** writes it. The command pushes to the direct base slice only. Run it again from the base slice to move the nudges one level further up.

The command shows the reason and changes nothing in these cases:

- The slice has no base slice.
- The base slice no longer exists.
- The base slices loop back on themselves.
- The slice holds no nudges of its own.
- The view is read-only.

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
  **entry** and **exit** rather than start and end, because a hop's start
  values describe the line arriving at it.
- An override for a hop the path no longer routes through is not shown.
  Overrides are kept by hop node, so one survives a path edit that removed
  its hop. The inspector ignores it, as the renderer does.

Click any row to switch the active slice to the one that set that value. The
popover and the dialog both close, and you land in the slice where changing or
clearing that value takes effect.

The inspector is read-only: it shows where a value lives and takes you there.
Change a value in the nudge panel's own controls, or use **Reset to inherited** for the active slice's own
override.

## Orientation

A model has no orientation of its own. The active data slice sets whether the
top-level containers run in rows or columns, and the model properties drawer
offers only the name and description.
