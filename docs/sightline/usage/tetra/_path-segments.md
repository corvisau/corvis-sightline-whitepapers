A channel or flow that routes through intermediate nodes carries a `path[]` of
hops. The line it draws is not one thing you nudge from its two ends. It is a
run of **segments**, and this is the surface for editing them.

A relationship's node sequence is `[nodeA, ...path hops, nodeB]`, so a path with
`n` hops has `n + 1` segments. A relationship with no path has exactly one,
whose two ends are the relationship's own, so a direct edge behaves here as it
always has.

Everything the popover writes goes to the **active data slice**, never to the
model. See [Per-Slice Layout & Nudge Overrides](/docs/sightline/usage/tetra/per-slice-layout) for the storage,
inheritance and reset rules that apply to every field below.

## Opening it

**Shift+right-click a channel or flow** on the canvas. The popover opens scoped
to the run of line you clicked, headed by the two nodes that segment joins:

```
< PLC 1 -> Sensor 1   2/3 >
```

The `<` / `>` buttons step to the neighbouring segment; the counter says where
you are. On a relationship with one segment the header names both
endpoints and there is no stepper.

While the popover is open, the segment it is editing glows on the canvas, so
the mapping from control to geometry is visible. The glow covers that segment
exactly; it stops at each node the line passes through.

### What a click resolves to

| Where you shift+right-click | What opens |
|---|---|
| On a segment's line | that segment |
| On a label pill | the segment the pill sits on |
| On the short run passing **through** an intermediate node | **nothing** — no segment owns that stretch |
| On a label pill that sits over one of those runs | the popover, on whichever segment it was already showing |
| Anywhere else on the edge (entry dot, arrowhead) | the popover, on whichever segment it was already showing |
| On a line that cannot be sliced at all (a `via`-routed relationship) | the popover, with a note saying the clicked segment could not be identified |

Opening nothing is deliberate. The run through a node is not part of any
segment, so no segment owns a click there.

A label pill counts as the line it sits on. A flow shows its id as a label by
default. `labelPositions` can place several pills along one relationship,
so a pill is often the easiest part of a long line to hit. Each one resolves
to the segment beneath it independently. The exception is a pill parked over a
run through a node. That run is not addressable, so the pill is not either, and
the popover opens on whatever segment it was already showing.

## What each control writes

The popover's two sections are **Start** (where the line leaves) and **End**
(where it arrives). Scoped to a named segment, those words are unambiguous. The
field they write to is not always the one they are named after. The
model names a hop's fields after the hop, not the segments touching it.

For segment `k` of a relationship with hops `h[0..n-1]`:

| Section | When | Writes |
|---|---|---|
| **Start** | `k == 0` | the relationship's `startEdge` / `startStandoff` / `startPort` |
| **Start** | `k > 0` | hop `h[k-1]`'s **exit**: `endEdge` / `endStandoff` / `endPort` |
| **End** | `k < n` | hop `h[k]`'s **entry**: `startEdge` / `startStandoff` / `startPort` |
| **End** | `k == n` | the relationship's `endEdge` / `endStandoff` / `endPort` |

So nudging the **Start** of the second segment of `A -> sw -> B` writes
`hops.sw.endEdge`, because the `end` geometry of `sw` is the line leaving it.

### Ports

Every end of every segment has a **Port** row, hop ends included. A port is a
slot relative to the centre of the edge the line attaches to. `0` is dead centre,
`+1` one step below/right, `-1` one step above/left.

On a hop the name inverts with everything else in the table above. A segment's
**Start** port is the hop's `endPort`, because leaving the hop is the hop's exit.

An explicit port on a hop overrides the router's own choice. Leave the port at its inherited value to keep the automatic behaviour.

### Via offset

`Via offset` is slice-only. Like the attach edges, standoffs and ports, it
is presentation and cannot be authored in a model file at either level. `via`
itself (which route the traffic borrows) stays model-side; `viaOffset` only says
where to draw the borrowed route.

`Via offset` is per-segment, not per-end: a hop's `via` follows another
channel's waypoints *for the segment arriving at that node*, so its `viaOffset`
belongs to that segment. The row addresses the hop the segment arrives at.

On the last segment there is no hop to store it on, so the row addresses the
relationship's own `viaOffset` and is labelled **Via offset (default)**. That
value is the default every `via` hop inherits, not the last segment's own.

## The run through a node

A multi-hop line enters each intermediate node on one port and leaves on
another, and the short run of line through the node joins the two. Those runs
are drawn separately from the segments and differ from them as follows:

|  | Real segment | Through-node run |
|---|---|---|
| Count for `n` hops | `n + 1` | `n` |
| Addressable (header, counter, `<`/`>`) | yes | **never** |
| Right-click resolves to it | yes | no — the click opens nothing |
| Glows | when it is the segment being edited | **never** |

A through-node run carries no hit target at all, so the node underneath it stays
clickable.

Because the runs are never counted or indexed, hiding them changes no segment
number and no click target. The next subsection describes how to hide them.

### Hiding them

**View → Hide Through-Node Runs** hides the runs, so a multi-hop line
reads as separate runs entering and leaving each node with a visible gap at each
one. It is a global view option, off by default, and persists on the data slice
alongside the other `hide*` toggles. PNG export honours it.

This is useful mainly on information flows, which render above nodes and so show their through-node runs
crossing the node box.

## Where routing is NOT edited

Routing is edited only in the segment popover, and it is slice-only. The props
drawer offers no attach edges, standoffs, ports or via offset, on the
relationship or on a path hop.

The drawer lists the path as **segment rows**, one per run of line between
`[nodeA, ...hops, nodeB]` (the same `n + 1` count as the popover). Each row is
labelled `From → To` with node names, using the popover's own arrow.

A row that arrives at a hop keeps that hop's structure: a node picker, a `via`
select, drag-reorder and remove. The final row, which arrives at `nodeB`,
carries its label only, since there is no hop there to edit.

Every row, the final one included, offers two more actions:

- **Split** breaks that row's run of line in two, inserting a hop with no
  node yet: splitting `A → B` gives `A → ?` and `? → B`.
- **Edit** opens the canvas segment popover on that row's run of line, the
  same popover a shift+right-click opens, scoped to that segment.

The picker, `via`, Split and drag-reorder write the path's structure directly,
which is model-side by design. Edit writes nothing itself: it opens the
popover, which stays the only place routing (`startEdge`/`endEdge`,
standoffs, ports, `viaOffset`) gets edited.

## Degradations worth knowing

- **A `via`-routed relationship has no glow.** Its waypoints are rebuilt from
  the route it borrows, so there is no geometry to slice.
  The popover cannot glow a segment it cannot locate. Shift+right-clicking its
  line still opens the popover, with a note that the clicked segment could not
  be identified. Its segments are otherwise fully editable, and stepping the
  header with `<` / `>` writes correctly. The drawer closes the rest of that
  gap: the Edit button on each segment row opens the popover on that exact
  segment. Every segment of a `via`-routed relationship is therefore
  reachable, though not by clicking the canvas. The per-hop attach edges and
  standoffs of a `via`-routed relationship never affect its geometry; only its
  via offsets do.
- **A hop that collapses into its neighbour renders no segment of its own.** Two
  hops inside one collapsed zone resolve to the same rendered node. The line
  therefore has fewer boundaries than the path implies, and any override on the
  collapsed hop stays inert until the zone expands again.
- **A node that appears twice as a hop shares one entry**, so both occurrences
  get the same routing. Overrides are keyed by hop node id, not by position.
- **A broken hop affects only that hop.** A hop naming an
  entity the model does not define is skipped, and its two neighbours join
  directly. A hop whose `via` channel does not exist keeps its place in the
  line. That one segment routes directly instead of along the missing
  channel. The rest of the path routes as authored. The connection is drawn
  with the routing warning, and the loader names the broken reference in the
  Problems panel. Only a path with no usable hop left collapses to a straight
  line between the relationship's own two ends.
