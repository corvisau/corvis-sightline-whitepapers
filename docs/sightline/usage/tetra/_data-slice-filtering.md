Tetra uses **bonsai filter expressions** to project a full hydrated model into a narrowed view called a _data slice_. This page covers the filter language, the execution pipeline and the Tetra sentinel transforms (`cascade()`, `exclude()`). It also covers set-pipeline programs, the prune and shade display modes, and YAML-level member filtering.

---

## DataSlice schema

A Tetra `DataSlice` has these fields:

| Field | Type | Notes |
|---|---|---|
| `id` | `string` | Opaque generated id (`s-xxxxx`) for new/duplicated slices; unchanged when editing an existing slice |
| `name` | `string` | Human-readable label |
| `description` | `string?` | Optional prose description |
| `filter` | `string \| string[] \| undefined` | Bonsai expression(s); absent/empty = full model |
| `viewOptions` | `DataSliceViewOptions?` | Saved view perspective (zoom, pan, display settings) |

Tetra has no `aggregationId` field, and it ignores one on a slice pasted from Lamina.

### Creating, editing, and duplicating slices

The toolbar's **Data slice** control appears in both the Diagram and the Table view. Its label reads **Data slice:** followed by the name of the active slice, or **None — full model** when no slice is active.

Click the control to open the slice popover. The popover lists **None — full model** and every slice. A slice that builds on a base slice appears indented under that base. Select the base itself from its own row. Click a slice to apply it and close the popover. The Up and Down arrow keys, Home and End move between the rows, and Escape closes the popover.

Below the list, choose an action:

- **New** opens an empty editor.
- **Edit** opens the currently-active slice with every field pre-filled; its id never changes.
- **Duplicate** opens the same editor pre-filled from the currently-active slice (name, filter, base slice, description) but under a fresh id. Saving creates an independent copy and leaves the source slice untouched.
- **Delete** asks for confirmation, then removes the active slice.

Slice ids are opaque and generated; the editor does not derive them from the typed name.

A **Show code** button in the top-right corner of the query builder reveals the exact bonsai program as read-only text. The program is the same statement list described below, and a **Copy** button copies it. Use it to debug a filter or to share it outside the editor.

### Include related types

The editor's Query Builder shows an **Include related** checkbox row above the filter builder. Ticking a type appends one combined statement to the program (`model |> where("item.kind |> oneOf('device', ...)") |> add()`), preceded by an auto-generated `// Pulling devices into the model` summary comment. Unticking a type removes it. Clearing every type removes the generated block entirely. The selectable types are `channel`, `flow`, `annotation`, `system`, `device`, `zone`, `network`, `group`, `information`.

---

## Filter expression language (bonsai)

Filters are written in [bonsai-js](https://github.com/bonsai-js/bonsai-js), a small expression language with pipe-transform syntax.

### Evaluation context

Each entity is evaluated with two aliases:

```
{ item: entity, [kind]: entity }
```

`kind` is the structural kind of the entity. So `item.tags` and `device.tags` both work on a device. This is the default Tetra context; it differs from Lamina, which uses type-prefixed context only (`cause.`, `event.`, etc.).

The `flow.` and `channel.` aliases work in a program filter, not in a legacy string filter. A `{ program }` filter gives every entity a `kind`, so `flow.tags` and `channel.tags` resolve. A legacy string filter gives only containment-tree nodes a `kind`. A flow or channel there has none, so `flow.tags` resolves to `undefined` and the expression matches nothing. Use `item.` as the alias that works in both.

### `item.kind` and `item.type`

`item.kind` selects the structural kind of an entity. `item.type` holds the entity's own domain type, where it has one.

| Entity | `item.kind` | `item.type` |
|---|---|---|
| Zone | `zone` | The zone's `zoneType`, if set |
| Group, device, network | `group`, `device`, `network` | Not set |
| Information item | `information` | The item's `informationType` (`system`, `function` or `security`) |
| Channel | `channel` | The channel `type`, for example `network_ip` |
| Flow | `flow` | The flow `type` (`system`, `function` or `security`) |
| Annotation | `annotation` | The annotation `type` |
| System | `system` | The system's own `type`, if set |

To select the flows of type `system`, combine the two: `item.kind == "flow" && item.type == "system"`. A bare `item.type == "system"` selects both system-typed flows and system-typed information items.

### Basic expressions

```
item.tags |> hasAny("ami")
device.tags |> hasAll("prod", "ami")
item.kind == "network"
item.id |> startsWith("DX-")
item.system == "AMI" && item.tags |> hasAny("ext")
```

The `|>` operator pipes the left-hand value into the right-hand transform as its first argument. All comparison and logical operators (`==`, `!=`, `&&`, `||`, `!`) work normally.

### Multi-line filters (AND semantics)

When `filter` is an array of strings, or when the edit dialog has multiple lines, the lines are AND-joined at evaluation time.

```yaml
filter:
  - item.tags |> hasAny("ami")
  - item.kind == "device"
```

Equivalent to `item.tags |> hasAny("ami") && item.kind == "device"`. Use multiple lines for readability; use `&&` within a single line to AND conditions that share a prefix.

### OR semantics

Separate lines are always AND-joined. To match either of two conditions, write them with `||` in a single expression:

```
item.system == "AMI" || item.system == "SCADA"
```

---

## Core transform functions

All operate on the piped value as their first argument.

| Transform | Signature | Description |
|---|---|---|
| `hasAny(…targets)` | `(arr, ...targets) => bool` | True if `arr` contains any of the targets |
| `hasAll(…targets)` | `(arr, ...targets) => bool` | True if `arr` contains all of the targets |
| `hasNone(…targets)` | `(arr, ...targets) => bool` | True if `arr` contains none of the targets |
| `hasExact(…targets)` | `(arr, ...targets) => bool` | True if `arr` contains exactly these targets and nothing else |
| `hasTags(…targets)` | alias for `hasAll` | |
| `isEmpty()` | `(arr) => bool` | True if `arr` is null, empty array, or empty string |
| `allOf(predicate)` | `(arr, pred) => bool` | True if every element in `arr` matches the predicate |
| `anyOf(predicate)` | `(arr, pred) => bool` | True if any element in `arr` matches the predicate |
| `noneOf(predicate)` | `(arr, pred) => bool` | True if no element in `arr` matches the predicate |

The bonsai stdlib also provides string transforms such as `startsWith`, `endsWith` and `includes`, for example `item.id |> startsWith("DX-")`.

### Address-semantic transforms

All three take a network's `addressing` list as the piped value and read both entry forms: the bare CIDR string and the `{ range, … }` object. `item.addressing` itself is the authored YAML, so `anyOf(.range == …)` matches object entries only. See [Network Addressing](/docs/sightline/usage/tetra/network-addressing#raw-entries).

| Transform | Signature | Description |
|---|---|---|
| `covers(…candidates)` | `(addressing, ...candidates) => bool` | True if any declared range **contains** any candidate. A bare address is read as a single host, so no `/32` is needed |
| `overlaps(…ranges)` | `(addressing, ...ranges) => bool` | True if any declared range shares address space with any argument. Symmetric, where `covers` is directional |
| `isTemplated()` | `(addressing) => bool` | True if any declared range carries a `{placeholder}` |

```
model |> where("item.addressing |> covers('10.1.2.5')") |> add()
model |> where("item.addressing |> overlaps('10.1.0.0/16')") |> add()
model |> where("item.kind == 'network' && item.addressing |> isTemplated()") |> add()
```

All three are **total**. Any input they cannot read yields `false` rather than an error. Unreadable input includes a missing `addressing`, a value that is not a list, an unparseable range or candidate, and an IPv6 argument. An unparseable value on either side is skipped. See [Network Addressing](/docs/sightline/usage/tetra/network-addressing) for the `addressing` field itself.

The Tabular Editor's `[Filter]` Basic tab offers the same transforms on the `addressing` field as the operators `covers`, `overlaps` and `is templated`. Each of the first two takes a single value; a call with several arguments stays in the Advanced tab.

---

## Sentinel transforms: `cascade()` and `exclude()`

These two transforms act as **end-of-pipe markers** that change how a matched entity is handled, as well as whether it passes.

### Default behaviour ("include")

A plain line with no sentinel:

```
item.tags |> hasAny("ami")
```

Matched nodes are kept; unmatched nodes are pruned. Ancestor containers of kept nodes are always retained so the tree stays coherent.

### `cascade()` — expand direct connectors

```
item.tags |> hasAny("ami") |> cascade()
```

When appended to a line, `cascade()` also pulls in every entity directly connected to the matched entities through `nodeA`, `nodeB`, `path`, `connections` or `devices`, one hop deep. The `connections` field on a device lists the networks, channels and flows the device takes part in.

> **Tetra caveat:** Tetra always pulls in the channels, flows and networks directly connected to a kept node, so `|> cascade()` has no effect in Tetra. In Lamina it selects the chains to expand. The auto-cascade toggle appends it for compatibility with Lamina.

The **auto-cascade toggle** in the data slice edit dialog appends `|> cascade()` to every expression line automatically. The dialog strips the suffix for display and adds it back on save. Lamina's dialog behaves the same way.

### `exclude()` — subtract after cascade

```
item.kind == "network" && item.tags |> hasAny("mgmt") |> exclude()
```

`exclude()` removes the matched entities from the result set. It runs after cascade expansion, so you can use it to trim nodes the cascade dragged in.

**Exclude wins over include:** if an entity is matched by both a plain line and an exclude line, it is gone.

```
# Keep all ami-tagged devices and their connectors...
item.tags |> hasAny("ami") |> cascade()
# ...but remove any device with the "legacy" tag that cascade pulled in
item.tags |> hasAny("legacy") |> exclude()
```

---

## The filter execution pipeline

A filter takes a model and returns a narrowed copy. It never changes the original. These rules decide what remains:

- Every node that matches the filter stays, together with its ancestor containers.
- Every channel, flow and network directly connected to a kept node is pulled in.
- A channel or flow with no surviving end is dropped. A channel with several targets survives when at least one target survives.
- An invalid filter returns the full model.

---

## Set-pipeline programs: the `follow` op

The legacy path above compiles a bonsai _predicate_ and cascades. The opt-in
`{ program: string[] }` filter shape instead runs an explicit **set-pipeline**.
Each statement reshapes a working
set, with tee ops (`add` / `remove` / `save`) writing to a result register:

```
model |> where("item.kind == \"device\"") |> add()
```

### Tree node inclusion — `information` badges follow the same rule as all other nodes

Tree nodes (zones, devices, groups, and `information` badges) survive into the
output only if their own id is present in the program's `result` register.
They also survive when the automatic upward ancestor-scaffolding reaches them;
it keeps each kept node's ancestors so the tree stays structurally valid. A kept device
does not automatically pull in its `information` badges. A badge must
appear in `result` in its own right (for example via `follow("children")` or
`add()`) to survive. Therefore `remove()` can target `item.kind == 'information'`:

```
model |> add()
model |> where("item.kind == 'information'") |> remove()
```

The above produces a model with all devices and zones intact but no information badges.

### `follow("edge"[, hops])` — walk one edge

`follow` reads one named edge field off each entity in the current set and
returns the reached targets. The
seeds themselves are excluded.

- **`edge`**: a single field name (string), for example `parent`, `children`, or a
  flow or channel endpoint field (`nodeA`, `nodeB`, `path`).
- **`hops`**: an optional number, the traversal depth (how many times to walk the
  edge). Omit it to walk to fixpoint. `follow("parent", 1)` walks one hop only.

```
D |> follow("parent") |> add()          # walk up to the root
A |> follow("parent", 1) |> follow("children", 1) |> add()   # 1-hop neighbourhood
```

> **`hops` is depth, not a list of fields.** `follow` takes exactly one edge
> name and an optional numeric depth. Several edge names, such as
> `follow("nodeA", "nodeB", "path")`, do not work. To follow several fields, see
> the Including all of a flow's hop-nodes section below.

### `removeRecursiveDown()` — remove a set and its whole children subtree

A shorthand for the two-statement pattern below, scoped to a whole subtree instead of
one entity type. Removes the piped set from `result`, then walks their
`children` edge to fixpoint and removes every reached descendant too:

```
model |> add()
X |> removeRecursiveDown()
```

is equivalent to:

```
model |> add()
X |> remove()
X |> follow("children") |> remove()
```

It takes no arguments. Unlike `follow()`, the edge is always `children`,
and there is no `hops` depth limit, so it always removes the full subtree.
It always targets the reserved `result` register and has no `add()`- or
`remove()`-style register argument. A subtree removal beyond this exact shape needs the
manual `follow("children")` + `remove()` form above.

### Including all of a flow's hop-nodes

A flow touches its endpoints (`nodeA`, `nodeB`) and every `path` waypoint. To
pull all of them in, tee each field separately from the same flow set. Do
not chain them, because chaining follows the next field off the previous field's
targets, not off the flow:

```
F |> follow("nodeA") |> add()
F |> follow("nodeB") |> add()
F |> follow("path")  |> add()
```

---

## Filter display modes: prune vs shade

The View popover carries a Prune tab and a Shade tab. The selected tab is the display mode, and it decides what the slice filter does with the entities it did not match.

| Mode | What happens to a non-matching entity |
|---|---|
| **Prune** | Tetra removes it. Only the matching subtree renders. |
| **Shade** | It stays on the canvas at 30% opacity. The model is unchanged. |

Shade mode suits orientation. The full diagram stays on screen with the non-matching entities faded, which keeps the surrounding context in view. An invalid filter returns an empty shaded set.

### The category rows on each tab

Both tabs list the same five categories with the same `Keep` and `Hide` buttons. `Hide` always removes, and the tab decides which entities it removes.

In shade mode the filter has split the canvas into two layers, and each tab owns one:

| Tab | Governs | A `Hide` on it removes |
|---|---|---|
| **Prune** | The entities the filter matched, at full opacity | That category, from the matched entities only |
| **Shade** | The entities the filter dimmed | That category, from the dimmed entities only |

Both sets act at the same time, on complementary halves. Take four flows, where `F1` and `F2` match the filter and `F3` and `F4` do not:

| Mode | Prune tab | Shade tab | Result |
|---|---|---|---|
| Shade | Flows = Hide | Flows = Keep | `F1` and `F2` go; `F3` and `F4` stay, dimmed |
| Shade | Flows = Keep | Flows = Hide | `F1` and `F2` stay at full opacity; `F3` and `F4` go |
| Shade | Flows = Hide | Flows = Hide | All four go |
| Prune | Flows = Hide | Either | All four go |

In prune mode nothing is dimmed, so the Shade tab has no second layer to act on and is inert.

### What each category covers

- **IP Channels** are channels of type `network_ip`. An untyped channel counts as one; a hosting or serial channel never does.
- **Flows** covers every flow.
- **Networks** removes the network node and its subtree.
- **Annotation Overlay** takes its layer from what the annotation links. An annotation is dimmed when every entity it points at is dimmed.
- **Through-Node Runs** is geometry rather than an entity, so it follows the layer of the edge that carries it. Removing a run moves no segment index and no hit target.

### Persistence and the no-slice case

The mode and both category sets save onto the data slice. The tabs render only while a data slice is active. When `None` is selected, the popover shows the Prune set alone as one flat list, because both sets use the same nouns and would otherwise appear as five pairs of identical rows.

---

## YAML-level member filtering

Independent of data slice filters, individual container `members:` entries in YAML support two filtering forms resolved at hydration time.

### FilteredMemberRef — prune a resolved container's subtree

```yaml
members:
  - id: z-it
    filter: 'item.tags |> hasAny("ami")'
```

Resolves the container `z-it` by ID, then prunes its subtree using the bonsai expression. Only nodes matching the filter survive within that container. The UI renders a **funnel badge** (⊻ icon) on the container node to indicate that a subtree filter applies. Hover over the badge to see the expression.

Schema: `{ id: string, filter: string }`; both fields are required.

The Edit drawer's **Member Filter** field, in a zone's or group's Advanced section, takes the same expression.

### ExprRef — resolve members dynamically by expression

```yaml
members:
  - expr: 'item.kind == "device" && item.tags |> hasAll("ami")'
```

Evaluated against every entity in the model at hydration time. All matching entities are inserted as members. Useful for dynamic groupings that do not need explicit ID lists.

Schema: `{ expr: string }`.
