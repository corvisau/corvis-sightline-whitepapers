A channel or flow that routes through intermediate nodes carries a path of
hops. The line it draws is not one thing you nudge from its two ends. It is a
run of **segments**, and this is the surface for editing them.

A relationship's node sequence is its first node, then its path hops, then its
last node. So a path with `n` hops has `n + 1` segments. A relationship with no path has exactly one,
whose two ends are the relationship's own.

Everything the popover writes goes to the **active data slice**, never to the
model. See [Per-Slice Layout & Nudge Overrides](/docs/sightline/usage/tetra/per-slice-layout) for the storage,
inheritance and reset rules that apply to every field below.

## Opening it

**Alt+right-click a channel or flow** (Option+right-click on a Mac) on the canvas. The popover opens scoped
to the run of line you clicked, headed by the two nodes that segment joins:

```
< PLC 1 -> Sensor 1   2/3 >
```

The `<` / `>` buttons step to the neighbouring segment; the counter says where
you are. On a relationship with one segment the header names both
endpoints and there is no stepper.

Drag the title bar to move the popover. It then reopens where you left it. See [Moving a nudge panel](/docs/sightline/usage/shared/annotations#moving-a-nudge-panel).

While the popover is open, the segment it is editing glows on the canvas, so
the mapping from control to geometry is visible. The glow covers that segment
exactly; it stops at each node the line passes through.

### What a click resolves to

| Where you Alt+right-click | What opens |
|---|---|
| On a segment's line | that segment |
| On a label pill | the segment the pill sits on |
| On the short run passing **through** an intermediate node | **nothing**; no segment owns that stretch |
| On a label pill that sits over one of those runs | the popover, on whichever segment it was already showing |
| Anywhere else on the edge (entry dot, arrowhead) | the popover, on whichever segment it was already showing |
| On a line that cannot be sliced at all (a route that lost a hop, or fell back to a direct line) | the popover, with a note saying the clicked segment could not be identified |

Opening nothing is deliberate.

A label pill counts as the line it sits on. A flow shows its id as a label by
default. The label positions can place several pills along one relationship,
so a pill is often the easiest part of a long line to hit. Each one resolves
to the segment beneath it independently. The exception is a pill parked over a
run through a node. That run is not addressable, so the pill is not either, and
the popover opens on whatever segment it was already showing.

## What each control writes

The popover's two sections are **Start** (where the line leaves) and **End**
(where it arrives). Each section has an **Offset** row, an **Edge** row and a
**Port** row. Scoped to a named segment, Start and End are unambiguous. The
value they change belongs to the node at that end of the segment. A hop therefore holds
two sets of values: one for the line arriving at it, and one for the line leaving it.

For segment `k` of a relationship with hops `h[0..n-1]`:

| Section | When | Changes the values of |
|---|---|---|
| **Start** | `k == 0` | The relationship's own start |
| **Start** | `k > 0` | Hop `h[k-1]`, for the line leaving it |
| **End** | `k < n` | Hop `h[k]`, for the line arriving at it |
| **End** | `k == n` | The relationship's own end |

So nudging **Start** on the second segment of `A -> sw -> B` changes the line
leaving `sw`.

### Ports

Every end of every segment has a **Port** row, hop ends included. A port is a
slot relative to the centre of the edge the line attaches to. `0` is dead centre,
`+1` one step below/right, `-1` one step above/left.

An explicit port on a hop overrides the router's own choice. Leave the port at its inherited value to keep the automatic behaviour.

### Via offset

**Via offset** is stored on the data slice. The model keeps which route the traffic
borrows. **Via offset** only says where to draw the borrowed route.

**Via offset** is per-segment, not per-end: a hop's via route follows another
channel's waypoints *for the segment arriving at that node*. So the offset
belongs to that segment. The row addresses the hop the segment arrives at.

Only a segment that follows another channel's route has the row. A segment that
arrives at a hop with no via route, and the last segment, are drawn direct and are
never offset. The panel therefore shows no **Via offset** row for them.

The row has an **X** and a **Y** control. Each has a minus and a plus button that
move the offset by 10 px, and a field where you type an exact value. Every change
stores both values for the segment.

A relationship can also carry its own via offset, the default every via segment
inherits. A via segment with no offset of its own shows that default and
labels the row **Via offset (from the relationship)**. A segment with no offset
anywhere shows **Via offset**. Changing a value stores it on that segment's hop
only, and the other via segments keep the default. When a data slice supplies the
value, the row also says which slice it comes from. **Reset to inherited** removes
the segment's own offset.

### Label

The **Label** section lists one row for each label pill on the relationship. A
relationship with no rows shows one label at the centre of the line.

Each row has an anchor, a value with **-** and **+** buttons, and a remove button.
Choose the anchor to set what the value measures:

| Anchor | The value measures |
|---|---|
| S | Pixels in from the start of the line |
| E | Pixels in from the end of the line |
| C | Pixels from the centre of the line; a negative value moves towards the start |
| % | A percentage of the routed length, from 0 to 100 |

The **-** and **+** buttons change the value by 10 pixels, or by 5 for a percentage.
You can also type a value. **Add** appends a row. A value past the end of the line
places the pill at that end. When the route grows or shrinks, a % label keeps its
place along the line, and the other anchors keep their distance.

Pills that touch merge into one pill. Changing the anchor of a row keeps its value,
limited to the range the new anchor allows.

Label positions are stored as a list. An entry is either a number or an anchor with
a value:

```yaml
labelPositions:
  - 20
  - { anchor: percent, value: 75 }
```

A positive number counts pixels from the start, a negative number counts pixels from
the end, and `0` is the centre. The rows show these as S, E and C. Editing the rows
rewrites the whole list with an anchor on every entry.

## The run through a node

A multi-hop line enters each intermediate node on one port and leaves on
another. The short run of line through the node joins the two. Those runs
are drawn separately from the segments and differ from them as follows:

|  | Real segment | Through-node run |
|---|---|---|
| Count for `n` hops | `n + 1` | `n` |
| Addressable (header, counter, `<`/`>`) | yes | **never** |
| Right-click resolves to it | yes | no; the click opens nothing |
| Glows | when it is the segment being edited | **never** |

A through-node run carries no hit target at all, so the node underneath it stays
clickable.

Because the runs are never counted or indexed, hiding them changes no segment
number and no click target.

### Hiding them

**View → Hide Through-Node Runs** hides the runs. A multi-hop line then
reads as separate runs entering and leaving each node, with a visible gap at each
one. It is a global view option, off by default, and persists on the data slice
alongside the other hide toggles in the View menu. PNG export honours it.

This is useful mainly on information flows, which render above nodes and so show their through-node runs
crossing the node box.

## Where routing is NOT edited

Routing is edited only in the segment popover, and it is slice-only. The props
drawer offers no attach edges, offsets, ports or via offset, on the
relationship or on a path hop.

The drawer lists the path as **segment rows**, one per run of line between
the relationship's first node, its hops and its last node. The count matches the popover:
`n + 1` for `n` hops. When the last hop is the last node itself, it folds into the final row,
giving `n` rows.
Each row is labelled `From → To` with node names, using the popover's own
arrow.

A row that arrives at a hop keeps that hop's structure: a node picker, a **via**
select, drag-reorder and remove. The final row, which arrives at the last node, has
no picker, drag or remove, since there is no hop to edit. It does offer a **via**
select: choosing a channel routes the last run along that channel, and
choosing `none (direct)` clears it. A path that already ends at the last node shows
one final row. It is not a hop row followed by a row from that node to itself.

Every row, the final one included, offers two more actions:

- **Split** breaks that row's run of line in two, inserting a hop with no
  node yet: splitting `A → B` gives `A → ?` and `? → B`. Picking a node for a hop
  sets its **via** to `any`, so the line follows the existing channels. This
  applies to a new hop and to a hop whose node you change. Choose `none (direct)`
  to draw that run directly.
- **Edit** opens the canvas segment popover on that row's run of line, the
  same popover an Alt+right-click opens, scoped to that segment.

The picker, **via**, Split and drag-reorder change the path's structure in the model
itself. Edit changes nothing itself: it opens the popover, which stays the only
place routing (attach edges, offsets, ports and via offset) gets edited.

## Degradations worth knowing

- **A via-routed relationship glows like any other, unless it lost a hop.** It
  opens on the clicked segment and glows that segment. When a hop has no
  position, or a via route cannot be found, the line follows the hops that
  remain and no longer matches the path, so there is no geometry to slice and
  no glow.
  The popover cannot glow a segment it cannot locate. Alt+right-clicking its
  line still opens the popover, with a note that the clicked segment could not
  be identified. Its segments are otherwise fully editable, and stepping the
  header with `<` / `>` writes correctly. The drawer closes the rest of that
  gap: the Edit button on each segment row opens the popover on that exact
  segment. Every segment of such a relationship is therefore reachable,
  though not by clicking the canvas. The per-hop attach edges and
  offsets of a via-routed relationship never affect its geometry; only its
  via offsets do. The one exception is an end on an information badge. The
  **Edge** row of that end sets the side of the badge the line leaves from:
  top, right, bottom or left. Without a chosen side, the line leaves the side
  that faces the first or last via route.
- **A hop that collapses into its neighbour renders no segment of its own.** Two
  hops inside one collapsed zone resolve to the same rendered node. The line
  therefore has fewer boundaries than the path implies, and any override on the
  collapsed hop stays inert until the zone expands again.
- **A node that appears twice as a hop shares one entry**, so both occurrences
  get the same routing. A slice keeps overrides by hop node, not by position.
- **A broken hop affects only that hop.** A hop naming an
  entity the model does not define is skipped, and its two neighbours join
  directly. A hop whose via channel does not exist keeps its place in the
  line. That one segment routes directly instead of along the missing
  channel. The rest of the path routes as authored. The connection is drawn
  with the routing warning. Tetra names the broken reference in the
  Problems panel, or the `path-hop-missing` and `path-via-missing` rules do when
  the model lists the starter package. Only a path with no usable hop left
  collapses to a straight line between the relationship's own two ends.
