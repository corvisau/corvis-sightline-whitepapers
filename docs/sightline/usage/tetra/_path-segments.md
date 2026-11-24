A channel or flow that routes through intermediate nodes carries a path of
hops. The line it draws is not one thing you nudge from its two ends. It is a
run of **segments**, and this is the surface for editing them.

A relationship's node sequence is its first node, then its path hops, then its
last node. So a path with `n` hops has `n + 1` segments. A relationship with no path has exactly one,
whose two ends are the relationship's own.

Everything the popover writes goes to the **active data slice**, never to the
model. See [Per-Slice Layout & Nudge Overrides](/docs/sightline/usage/tetra/per-slice-layout) for the storage,
inheritance and reset rules that apply to every field below.

## Opening It

**Alt+right-click a channel or flow** (Option+right-click on a Mac) on the canvas. The popover opens scoped
to the run of line you clicked. Its title bar reads **Layout**, like the container and annotation panels. A header names the two nodes that segment joins:

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

### Alt+right-Click Targets

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

## Segment Popover Controls

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

### Via Offset

**Via offset** is stored on the data slice. The model keeps which route the traffic
borrows. **Via offset** only says where to draw the borrowed route.

**Via offset** is per-segment, not per-end: a hop's via route follows another
channel's waypoints *for the segment arriving at that node*. So the offset
belongs to that segment. The row addresses the hop the segment arrives at.

Only a segment that follows another channel's route has the row. That includes a
segment left on the `auto` setting of the via select, when a channel is drawn
between its two nodes. A segment set to `direct` is drawn direct and is never
offset. The same holds for a segment with no channel between its two nodes. Neither
has a **Via offset** row. The last segment follows the same rule.

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

## A Flow with Several End Nodes

A flow can list several nodes in `nodeB`. The model keeps one flow, and the canvas draws one line for each node in the list:

```yaml
flows:
  - id: flow-telemetry
    nodeA: device-rtu
    nodeB: [device-historian, device-scada]
    path:
      - network-control
```

Every line runs from `nodeA` along the same `path` to its own end node. The path is drawn once, and each end node adds a line that branches off at the last hop. The model tree, the Tabular Editor and the properties drawer show the flow once. Clicking any line selects the flow, and **Edit…** on any line opens its drawer. The drawer lists the end nodes as rows, where a node is added, removed or moved. The first row is the End row of the Path section.

Each line has its own segments. A segment that belongs to the shared path edits all the lines at once. A segment that arrives at an end node edits that line only:

| Segment | Where it is stored | Lines it changes |
|---|---|---|
| Any segment that starts or ends at a hop of the path | The flow | All |
| The last segment, which ends at an end node | The line you clicked | One |

Alt+right-click the line you want to change. The popover header names the two nodes of the segment. A data slice that includes the flow shows every line, and a filter that matches one end node shows the line to that node. To give each end node its own route, define one flow for each route.

## A Channel with Several End Nodes

A channel can also list several nodes in `nodeB`. A hypervisor that hosts several virtual machines is the usual case. The model keeps one channel, and the canvas draws one line from `nodeA` to each node in the list, styled alike:

```yaml
channels:
  - id: channel-hosting
    channelType: hosting
    nodeA: device-hypervisor
    nodeB: [device-vm-historian, device-vm-scada]
```

The model tree, the Tabular Editor and the properties drawer show the channel once. Clicking any line selects the channel, and **Edit…** on any line opens its drawer. An annotation linked to the channel is placed on its first line, and a glow on the annotation lights every line.

A hosting channel with several end nodes is several independent hosting links. A network or virtual channel with several end nodes stays one shared medium in the model, and the canvas draws it as separate lines. Rules and reports still see every end node of the channel.

A `via` in a flow path that names the channel follows the line between the two nodes of that run. If no line of the channel joins the two nodes, the run is drawn direct. Two end nodes of one channel do not reach each other.

A channel that has a `path` shares it between its lines. A segment that belongs to the shared path edits all the lines at once, and the segment that arrives at an end node edits that line only, as for a flow. Alt+right-click the line you want to change.

## The Run Through a Node

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

### Hiding Them

**View → Hide Through-Node Runs** hides the runs. A multi-hop line then
reads as separate runs entering and leaving each node, with a visible gap at each
one. It is a global view option, off by default, and persists on the data slice
alongside the other hide toggles in the View menu. PNG export honours it.

This is useful mainly on information flows, which render above nodes and so show their through-node runs
crossing the node box.

## A Zone or Group as a Hop

A hop can be a zone or a group, as well as a device, a network or an
information item. The hop picker lists them beside the other nodes. Use one when
several channels share a leg that ends at a container and split inside it.

The line **lands** on the container. It meets the border of the box once, on one
port, and carries on from that port. The next hop is usually a node inside the
box, so the line turns inwards and does not cross the box to the far side. When a
hop is a channel that is drawn to the container, the line follows that channel to
the container border, and the next run starts where the channel ended.

When both neighbours of the container are outside it, the line crosses the box:
it enters on one side and leaves on another, as it does at a device.

- **A collapsed container is one box.** A hop on the container and a hop on a
  node inside it draw as one landing, with no run between them.
- **A hidden group has no drawn border.** A line that lands on one ends at the
  edge of its members, where no box shows.

## Routing Edit Scope

Routing is edited only in the segment popover, and it is slice-only. The props
drawer offers no attach edges, offsets, ports or via offset, on the
relationship or on a path hop.

The drawer lists the path as a sequence of nodes. A **Start** row names the
relationship's first node, each hop has a row of its own, and an **End** row names
its last node. Between each pair of rows sits a **connector**, the run of line that
joins them. The count matches the popover: `n` hops give `n + 1` connectors. When
the last hop is the last node itself, it folds into the End row, giving `n`
connectors.

A hop row has a node picker, a drag handle and a remove button. The picker lists
nodes only, and choosing one draws the run into it directly. The drag handle
reorders the hops. The remove button deletes the hop and joins the two connectors
beside it. The Start and End rows are fixed.

Every connector, the one into the End row included, has the same controls:

| Control | What it does |
|---|---|
| **via** | Selects `auto`, `direct` or a channel for the run |
| **Edit routing** | Opens the canvas segment popover on this run of line, the same popover an Alt+right-click opens |
| **Add hop** | Splits the run in two by inserting a hop with no node yet: `A → B` becomes `A → ?` and `? → B` |

The **via** select lists `auto`, `direct` and the channels drawn between the
connector's two nodes. A channel is drawn between two nodes when one is its first node
and the other is one of its last nodes. Choosing a channel makes the run follow it end
to end. Choosing `direct` draws the run straight between the two nodes.

`auto` is the default. The run follows a channel drawn between its two nodes, and draws
direct when there is none. Tetra prefers a channel that is not a hosting channel, then
takes the first in the model. The option names the channel it picks, for example
`auto (C-3KNT7)`, or reads `auto (direct)`. A flow uses `auto` on every connector,
including a flow with no hops, so a flow beside a channel rides that channel. A channel
uses `auto` only once its path has a hop. The one connector of a channel with no hops
lists `direct` and the other channels between its ends, because those are its own
parallel siblings.

Several relationships can share one leg this way. Give a channel that carries the
shared first leg, and each relationship's connector along it follows that channel. The
lines run together along it and split where it ends.

The select stays inert while the node at the end of its connector is not picked. A
channel never lists itself, so the select of a channel relationship offers only other
channels. The choice on the connector into the End row is stored on a trailing hop
that names the last node.

A path edited in the model file can hold a hop that names a channel that is not drawn
between the hop's two nodes. Tetra does not follow it: that run draws directly.
The connector shows the choice as a greyed-out option and leaves it as it is until you
choose another value. A `via` that holds a glob such as `*` does not load; see
[Compatibility](/docs/sightline/upgrade/compatibility).

The picker, the **via** select, **Add hop** and drag-reorder change the path's
structure in the model itself. **Edit routing** changes nothing itself: it opens the
popover, which stays the only place routing (attach edges, offsets, ports and via
offset) gets edited.

## Degradations Worth Knowing

- **A via-routed relationship glows like any other, unless it lost a hop.** It
  opens on the clicked segment and glows that segment. When a hop has no
  position, or a via route cannot be found, the line follows the hops that
  remain and no longer matches the path, so there is no geometry to slice and
  no glow.
  The popover cannot glow a segment it cannot locate. Alt+right-clicking its
  line still opens the popover, with a note that the clicked segment could not
  be identified. Its segments are otherwise fully editable, and stepping the
  header with `<` / `>` writes correctly. The drawer closes the rest of that
  gap: the **Edit routing** button on each connector opens the popover on that
  exact segment. Every segment of such a relationship is therefore reachable,
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
  directly. A hop whose via channel does not exist, or exists but is not drawn
  between the hop's two nodes, keeps its place in the line. That one segment
  routes directly instead of along that channel. The rest of the path routes as authored. The connection is drawn
  with the routing warning. Tetra names the broken reference in the
  Problems panel, or the `path-hop-missing` and `path-via-missing` rules do when
  the model lists the starter package. A channel that does not join the hop is
  always named by Tetra, because the rules check only that the channel exists. Only a path with no usable hop left
  collapses to a straight line between the relationship's own two ends.
