Tetra uses **bonsai filter expressions** to project the full model into a narrowed view called a _data slice_. This page covers the filter language, the execution pipeline and the Tetra sentinel transforms (`cascade()`, `exclude()`). It also covers set-pipeline programs, the prune and shade display modes, and the member filter fields of a zone or group.

---

## Creating, editing, and duplicating slices

A data slice has a name, an optional description, an optional query and a saved view perspective (zoom, pan and display settings). A slice with no query shows the whole model.

The toolbar's **Data slice** control appears in the Diagram view only. A data slice narrows the diagram, and the Table view always lists the whole model. The active slice stays selected while the Table view is open. The control's label reads **Data slice:** followed by the name of the active slice.

Tetra draws the diagram through a data slice. When a model has slices, one is always active: the slice you last used, or the first slice when that one no longer exists. To view the whole model, create a slice with no filter.

A slice saves its view perspective, orientation included. A slice that saved no orientation opens in rows. The **Layout** field in the model's properties does not change the diagram while a slice is active. On a model with no slices, it sets the orientation that the first new slice starts from.

In the web shell, the active slice belongs to the browser tab. It survives a reload and is lost when you close the tab, so a new tab opens on the model's first slice. The VS Code extension keeps the choice per workspace.

A model with no data slices shows no **Data slice** control. The Diagram view shows the message "This model has no data slices. Create one to draw it." and a **Create data slice** button, which opens the slice editor. The Table view lists the model as usual.

A user who cannot edit the model sees the message "The model author hasn't created a data slice yet." instead, with no button. A web shell tab opened with a reader token is one example.

Click the control to open the slice menu. A row of four buttons stays at the top of the menu: **New**, **Edit**, **Duplicate** and **Delete**. Below the buttons, the menu lists the slices and scrolls when the list is long, so the buttons never scroll out of view. A slice that builds on a base slice opens a submenu to the side. The first entry of the submenu is the base slice itself, followed by the slices that build on it. Click a slice to apply it and close the menu. The Up and Down arrow keys, Home and End move between the entries. The Right arrow key opens a submenu, the Left arrow key closes it, and Escape closes the menu.

The buttons at the top of the menu do these actions:

- **New** opens an empty editor.
- **Edit** opens the currently-active slice with every field pre-filled; its id never changes.
- **Duplicate** opens the same editor pre-filled from the currently-active slice (name, filter, base slice, description) but under a fresh id. Saving creates an independent copy and leaves the source slice untouched.
- **Delete** asks for confirmation, then removes the active slice. The first remaining slice becomes active. When you delete the last slice, the confirmation warns that the diagram stays empty until you create another one.

A new or duplicated slice appears in the popover and becomes active as soon as you save the editor. It reaches the model file with your other unsaved changes when you click **Save all**.

The **PNG**, **Legend (PNG)** and **Draw.io** exports capture the diagram. On a model with no data slices they are disabled until you create one. Reports stay available.

The editor shows the slice's query under a **Query** label as read-only text, with a line above it that reads **(no query)**, **1 statement** or the number of statements. Select **Edit query** to open the query builder in its own dialog. **Apply** returns the edited query to the editor, and **Cancel** or Escape keeps the query as it was. Escape closes only the query dialog.

A new slice has no query. A slice with no query shows the whole model. **Edit query** on a slice with no query opens the builder on `model |> add()`. A statement counts toward the result only when it ends in `add()`. The slice stays without a query until you select **Apply**. A statement that does not add anything, such as a `remove()` or a `where()` without `add()`, then acts on an empty result. The `remove()` examples in this page start from `model |> add()` for the same reason.

A **Show code** button in the top-right corner of the query builder, opened from **Edit query**, reveals the exact bonsai program as read-only text. The program is the same statement list described below, and a **Copy** button copies it. Use it to debug a filter or to share it outside the editor.

A statement in the query builder can be disabled and enabled again with the round button at the right of its row. A slice whose statements are all disabled shows the whole model, as a slice with no query does. See [Disabling a statement](/docs/sightline/usage/shared/tabular-filtering#disabling-a-statement).

### Filtering by kind

The **+ step** menu in the query builder, opened from **Edit query**, lists two steps under **Filters** that select items by kind. `includeKinds("zone", "flow")` keeps only the items of the listed kinds. `excludeKinds("information")` drops them and keeps everything else. Each step opens an editor with a checkbox for every kind: `channel`, `flow`, `annotation`, `system`, `device`, `zone`, `network`, `group` and `information`. Like `where()`, a kind step filters the current set, so end the statement in `add()`, `remove()` or `save()`:

```
model |> includeKinds("device", "zone") |> add()
model |> add()
model |> excludeKinds("information") |> remove()
```

`includeKinds("zone")` is the same as `where("item.kind |> oneOf('zone')")`. A slice saved with the former **Include related** checkboxes keeps working: the generated statement and its comments stay in the program as written, and the builder no longer edits them.

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
| Channel | `channel` | Not set. The channel `channelType` holds `network`, `virtual` or `hosting` ([channel types](/docs/sightline/methodology/tetra-channels-and-flows#channel-types)) |
| Flow | `flow` | The flow `type` (`system`, `function` or `security`; see [flow types](/docs/sightline/methodology/tetra-channels-and-flows#flow-types)) |
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

A tag filter matches the tag exactly, including its case. `hasAny("ami")` does not match a tag written `AMI`. To keep tags consistent, use the **Tags** field in the props drawer. As you type, the field lists the tags that other elements in the model already use. Matching ignores case, and tags already on the element are left out. Pick a suggestion to add it. To add a tag that no element uses yet, type it and press Enter.

### Protocol filters

A channel carries a `protocol` stack, and a flow carries the stack that it lists itself. These fields read a stack, or a value derived from the model, in a filter:

| Field | Holds | Example |
|---|---|---|
| `protocol` | The whole stack, bottom to top. A channel with no stack of its own holds the stack that it inherits from its network | `item.protocol \|> hasAny("ipsec")` |
| `protocolTop` | The top entry of the stack. An entity with an empty stack has none | `item.protocolTop == "dnp3"` |
| `topology` | `point-to-point` or `multipoint`, derived for a channel. A hosting channel has none | `item.topology == "multipoint"` |
| `hopProtocols` | The distinct protocols across the hops of a flow's route. A flow with no resolved hop has none | `item.hopProtocols \|> hasAny("ipsec")` |

`hasAny("ipsec")` selects every channel whose stack contains `ipsec`, wherever it sits in the stack. `protocolTop == "dnp3"` selects the channels and flows that carry DNP3 as their highest protocol.

`item.hopProtocols |> hasAny("ipsec")` selects the flows that cross a tunnel on some hop. The [protocol stack by hop](/docs/sightline/methodology/tetra-channels-and-flows#protocol-stack-by-hop) section says how a hop is found.

In the Tabular Editor, the Basic filter tab lists **Top of protocol stack**, **Topology** and **Protocols on any hop** beside the other fields.

### Multi-line filters (AND semantics)

When a slice's filter has more than one line, the lines are AND-joined at evaluation time. For example, the two lines `item.tags |> hasAny("ami")` and `item.kind == "device"` are equivalent to `item.tags |> hasAny("ami") && item.kind == "device"`. Use multiple lines for readability; use `&&` within a single line to AND conditions that share a prefix.

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

All three take a network's `addressing` list as the piped value and read both entry forms: the bare CIDR string and the `{ range, … }` object. `item.addressing` itself holds each entry as it was entered, so `anyOf(.range == …)` matches object entries only. See the [Network Addressing](/docs/sightline/usage/tetra/network-addressing#raw-entries) section for how the two entry forms are represented.

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

When appended to a line, `cascade()` also pulls in every entity directly connected to the matched entities, one hop deep. It reaches them through `nodeA`, `nodeB`, `path`, `connections` or `devices`. The `connections` field on a device lists the networks, channels and flows the device takes part in.

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

The string filter above is a bonsai _predicate_ that selects entities and cascades. The opt-in
`{ program: string[] }` filter shape instead runs an explicit **set-pipeline**.
Each statement reshapes a working
set, and `add()` and `remove()` write it into or out of the result:

```
model |> where("item.kind == \"device\"") |> add()
```

### Tree node inclusion — `information` badges follow the same rule as all other nodes

Tree nodes (zones, devices, groups, and `information` badges) survive into the
output only if they are in the program's result.
They also survive as the ancestors of a kept node, so the tree stays structurally valid. A kept device
does not automatically pull in its `information` badges. A badge must
appear in the result in its own right (for example via `follow("children")` or
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

### `includeKinds()` and `excludeKinds()` — filter by entity kind

`includeKinds("a", "b")` keeps the items whose `item.kind` is one of the listed kinds. `excludeKinds("a", "b")` keeps the items whose kind is none of them, including an item with no kind. Both take quoted kind names. `includeKinds()` with no kinds keeps nothing, and `excludeKinds()` with no kinds keeps everything. Neither adds to the result by itself.

### `removeRecursiveDown()` — remove a set and its whole children subtree

A shorthand for the two-statement pattern below, scoped to a whole subtree instead of
one entity type. Removes the piped set from the result, then walks their
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
A subtree removal beyond this exact shape needs the
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

- **IP Channels** are network channels whose protocol list includes `ip` or is empty. A channel with no list of its own counts the list that it inherits from its network. A channel with no protocol at all counts as one. The [channel types](/docs/sightline/methodology/tetra-channels-and-flows#channel-types) table says what each type means. A hosting channel, a virtual channel and a serial channel never do.
- **Flows** covers every flow.
- **Networks** removes the network node and its subtree.
- **Annotation Overlay** takes its layer from what the annotation links. An annotation is dimmed when every entity it points at is dimmed.
- **Through-Node Runs** is geometry rather than an entity, so it follows the layer of the edge that carries it. Removing a run moves no segment index and no hit target.

### Persistence and the no-slice case

The mode and both category sets save onto the data slice. The tabs render only while a data slice is active. On a model with no data slices, the popover shows the Prune set alone as one flat list. Both sets use the same nouns, so together they would appear as five pairs of identical rows.

---

## Member filters on a zone or group

Independent of data slice filters, a zone or group can carry two filters of its own. Tetra resolves them when it loads the model. Both fields are in the Advanced section of the zone or group props drawer.

### Member Filter: prune a container's subtree

**Member Filter** takes a bonsai expression, for example `item.tags |> hasAny("ami")`. Only the nodes in that container that match the expression survive. The canvas shows a **funnel badge** (⊻ icon) on the container node to indicate that a subtree filter applies. Hover over the badge to see the expression.

### Include by expression: add members dynamically

**Include by expression** takes a bonsai expression, for example `item.kind == "device" && item.tags |> hasAll("ami")`. Tetra evaluates it against every entity in the model when the model loads and inserts every match as a member. It suits dynamic groupings that do not need an explicit list of members.

Enter an expression and save to add every matching item to the container. Clear the field and save to remove the rule. Tetra refuses an expression it cannot compile and changes nothing.

The field edits one expression. A container with several such rules shows them in the field, read-only, joined by ` ; `, and keeps all of them when you save other changes. Editing those rules needs the model file, because the drawer offers no control for it.

Tetra does not store the matched items as members, because the expression adds them again on load.
