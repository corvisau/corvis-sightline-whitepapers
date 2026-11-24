Tetra uses **bonsai filter expressions** to project the full model into a narrowed view called a _data slice_. This page covers the filter language, the execution pipeline and the Tetra sentinel transforms (`cascade()`, `exclude()`). It also covers set-pipeline programs, how a slice displays, and the member filter fields of a zone or group.

---

## Creating, Editing, and Duplicating Slices

A data slice has a name, an optional description, an optional query and a saved view perspective (zoom, pan and display settings). A slice that has no query and no included entries from the Model Tree shows the whole model. A slice made with **New** starts empty instead; see [Slice selection from the Model Tree](#slice-selection-from-the-model-tree).

A data slice narrows the diagram, and the Table view always lists the whole model. The active slice stays selected while the Table view is open. The toolbar names the active slice, or reads "No slice open", and its **Close slice** button closes the slice in either view. The **Slices** view chooses the active slice and creates, edits, duplicates and deletes slices; see [Choosing slices from the Slices view](#choosing-slices-from-the-slices-view).

Tetra draws the diagram through a data slice, and a model opens with one already active. Tetra opens the slice you last had open in this model when it still exists. Otherwise it opens the slice highest in the **Slices** view. That is the slice with the fewest folders above it, and the first of those in the order the view lists them. A slice outside every folder therefore opens before a slice inside one.

After **Close slice**, or after you delete the active slice, the Diagram view and the Model Tree show the message "Open a slice in the Slices view to show the model." They keep the message until you open a slice in the **Slices** view. To view the whole model, make a slice whose query is `model |> add()`: select **Edit query** on a new slice and apply it.

A slice saves its view perspective, orientation included. A slice that saved no orientation opens in rows. The **Layout** field in the model's properties does not change the diagram while a slice is active. On a model with no slices, it sets the orientation that the first new slice starts from.

The VS Code extension remembers the active slice for the workspace, so a reload or a new window opens the same slice again. The web shell remembers it for the browser tab, so a reload opens the same slice and a new tab opens the highest slice. Closing the slice clears what is remembered, and the next reload opens the highest slice.

On a model with no data slices, the Diagram view shows the message "This model has no data slices. Create one to draw it." and a **Create data slice** button, which opens the slice editor. The Table view lists the model as usual.

A user who cannot edit the model sees the message "The model author hasn't created a data slice yet." instead, with no button. A web shell tab opened with a reader token is one example.

These actions work on slices:

- **New slice** opens an empty editor. The slice it creates starts empty: it shows nothing until you include entries from the Model Tree or apply a query.
- **Edit** opens the slice with every field pre-filled; its id never changes.
- **Duplicate** opens the same editor pre-filled from the slice (name, filter, description) but under a fresh id. Saving creates an independent copy and leaves the source slice untouched.
- **Delete** asks for confirmation, then removes the slice. When you delete the active slice, no slice is active until you open another. When you delete the last slice, the confirmation warns that the diagram stays empty until you create another one.

A new or duplicated slice appears in the **Slices** view and becomes active as soon as you save the editor. It reaches the model file with your other unsaved changes when you click **Save all**.

The **PNG**, **Legend (PNG)** and **Draw.io** exports capture the diagram. On a model with no data slices they are disabled until you create one. Reports stay available.

The editor shows the slice's query under a **Query** label as one summary line, for example **Filter · 1 statement · 41 items match**. The line has up to three parts:

- The type is **Filter** for a query that the Basic tab can show, **Adds** when every enabled statement only adds items, **Removes** when every enabled statement only removes them, and **Custom query** for any other query. A query with no enabled statement shows no type.
- The statement count reads **(no query)**, **1 statement** or the number of statements, and adds the number of disabled statements.
- The match count gives the size of the result over the model. It appears once the query has an enabled statement.

Select **Edit query** to open the query builder in its own dialog, where each statement shows as a step. **Apply** returns the edited query to the editor, and **Cancel** or Escape keeps the query as it was. Escape closes only the query dialog.

A new slice has no query and shows nothing, because it starts empty. An older slice that has no query and no included entries shows the whole model. **Edit query** on a slice with no query opens the builder on `model |> add()`. A statement counts toward the result only when it ends in `add()`. The slice stays without a query until you select **Apply**. A statement that does not add anything, such as a `remove()` or a `where()` without `add()`, then acts on an empty result. The `remove()` examples in this page start from `model |> add()` for the same reason.

A **Show code** button in the top-right corner of the query builder, opened from **Edit query**, reveals the exact bonsai program as read-only text. The program is the same statement list described below, and a **Copy** button copies it. Use it to debug a filter or to share it outside the editor.

A statement in the query builder can be disabled and enabled again with the round button at the right of its row. A slice whose statements are all disabled shows the whole model, as an older slice with no query does. A slice that has included entries from the Model Tree shows only those entries when its statements are all disabled. See [Disabling a statement](/docs/sightline/usage/shared/tabular-filtering#disabling-a-statement).

### Saving and Reverting the View

A change in the **View** menu applies to the diagram at once. The slice does not store it until you save it. The toolbar then shows two buttons until the view matches the stored view again:

- **Save view** stages the current view on the active slice. The change reaches the model file when you click **Save all**. A user who cannot edit the model sees this button disabled.
- **Revert view** discards the unsaved **View** menu changes and restores the view that the active slice stores. It stages nothing, so a user who cannot edit the model can use it.

If you click **Save view** and then change the view, **Revert view** restores the view that you staged. A slice that stores no view options reverts to the default view.

Collapse levels are not part of **Revert view**. Each collapse change is staged as you make it, and **Undo** reverses it.

### Filtering by Kind

The **+ step** menu in the query builder, opened from **Edit query**, lists two steps under **Filters** that select items by kind. `includeKinds("zone", "flow")` keeps only the items of the listed kinds. `excludeKinds("information")` drops them and keeps everything else. Each step opens an editor with a checkbox for every kind: `channel`, `flow`, `annotation`, `system`, `device`, `zone`, `network`, `group` and `information`. Like `where()`, a kind step filters the current set, so end the statement in `add()`, `remove()` or `save()`:

```
model |> includeKinds("device", "zone") |> add()
model |> add()
model |> excludeKinds("information") |> remove()
```

`includeKinds("zone")` is the same as `where("item.kind |> oneOf('zone')")`. A slice saved with the former **Include related** checkboxes keeps working: the generated statement and its comments stay in the program as written, and the builder no longer edits them.

---

## Choosing Slices from the Slices View

While a Tetra model is open, the activity bar holds a **Slices** view directly below the **Model Tree**. The web shell shows the same list as a **Slices** section below its Model Tree. The view lists the slices of the model in folders, with the folders first. The folders open collapsed, so a long list shows its folders and the slices outside any folder. A folder row shows a folder icon that opens and closes with the folder, a slice row shows a slice icon, and a vertical guide line runs beside each level of nesting. Typing in the filter field opens the folders that hold a match. It is the only control that changes the active slice.

- Click a row to highlight it. Hold Ctrl or Cmd and click to highlight several rows. Highlighting a row changes neither the diagram nor the Model Tree.
- Double-click a slice to open it and make it the active slice. A dot and bold text mark the open slice. A folder that holds the open slice shows a hollow circle, so the trail stays visible when the folder is collapsed. When the open slice changes, the view opens its folders and scrolls to the slice's row.
- **New slice** opens the editor for a new slice. A model with no slice shows "Create a slice to choose what the canvas shows." and this button.
- The icon buttons in the toolbar above the filter act on the highlighted rows: **Open** (or **Close slice** on the open slice), **Edit**, **Duplicate** and **Delete** for slices, and **Rename**, **New folder** and **Delete folder** for folders. Point at a button to see its name; a disabled button shows its reason too. Right-click a row, or press the context-menu key or Shift+F10 on it, for a menu of the same actions. A right-click on a row that is not highlighted highlights it first, and on a highlighted row it acts on the whole highlight. **Edit** and **Duplicate** open the slice editor, and **Delete** asks for confirmation. When the highlight holds both slices and folders, an action that one of them lacks shows disabled with the reason "Not every picked row supports this".
- **Close slice** leaves no slice active. **New slice** with one folder highlighted creates the slice in that folder.
- To make a folder, select **New folder** in the toolbar. A folder named New Folder appears (New Folder 2 if that name is taken), inside the highlighted folder when one is highlighted, and its name opens for editing. Type a name and press Enter, or select elsewhere, to keep it. Press Escape to cancel, which removes the folder. A folder name cannot be empty, cannot repeat, and cannot differ from a sibling folder only by case. The view shows the reason under the name and keeps the name open for editing.
- To move a slice, drag its row onto a folder. To move several, highlight them and drag one of them. Drag a folder onto another folder to nest it, with its subfolders and slices. Drop on the empty space under the rows to move to the top level. The folder under the pointer is outlined while it accepts the drop. A folder cannot move into itself or its own subfolders, or into a folder that already holds a folder of the same name (also by case); the view then shows the reason and changes nothing. A model that you cannot edit does not let you drag.
- **Rename** opens the name of the highlighted folder for editing. Press Enter to keep the new name, or press Escape to leave the folder as it was. The slices and subfolders in the folder move with it. The view refuses an empty name, a name with a slash, and a name that repeats a sibling or differs from it only by case. It shows the reason under the name and keeps the name open.
- **New folder** in a folder's row or menu adds the new folder inside that folder, as the toolbar button does with the folder highlighted.
- **Delete folder** works only when the folder holds no slice and no subfolder.

A model that you cannot edit shows its buttons and its menu items disabled, with the reason as the tooltip. **Open** and **Close slice** stay available, because they change no data. Moves and folder changes reach the model file with your other unsaved changes when you click **Save all**.

Each slice needs its own id. When the model file defines two slices with the same id, Tetra loads the first one and ignores the others. The Problems panel shows a warning for each ignored slice. Remove or rename the extra copy in the model file.

## Slice Selection from the Model Tree

The Model Tree opens in Browse. Browse lists only the rows that the active data slice shows, and it carries no toggle. Select **Build** at the left of the tree toolbar to choose what the slice shows. A model that you cannot edit always shows Browse. See [Build and Browse](/docs/sightline/usage/shared/tree-browser-panel#build-and-browse).

In Build, while a data slice is active, every row of the Model Tree carries a toggle. The toggle chooses what the slice shows. The tree lists every row of the model. Nothing greys out for being outside the slice, because a row has to stay reachable to be turned on.

The channels, flows, systems and annotations sit under the **Channels**, **Flows**, **Systems** and **Annotations** roots. Each entity there has a toggle. A root has none.

Flows, channels and annotations can sit in folders under their root. A folder row has a toggle too. The toggle sets the same state on every entity inside the folder, at every depth, and the slice lists those entities and never the folder. The folder row shows a state only when all of its entities share it. See [Folders for flows, channels and annotations](/docs/sightline/usage/shared/tree-browser-panel#folders-for-flows-channels-and-annotations).

With no slice active, the Model Tree lists no rows. It shows the message "Open a slice in the Slices view to show the model."

A row is in one of four states. A row can also be in the slice without a state of its own. The mark and the row text both show the state, so you can read the tree at a glance. A blue mark means you set the row. A grey mark means something else selected it. A maroon mark means you excluded it.

| State | Mark | Row text | Effect |
|---|---|---|---|
| Not set | Empty box with a dashed outline | Grey | The slice does not name the row. |
| Included | Blue box with a tick | Normal | The slice shows the row. |
| Included with contents | Blue box with a down arrow | Normal | The slice shows the row and everything it contains. |
| Excluded | Maroon box with a cross | Grey, struck through | The slice does not show the row, whatever else selects it. |
| Included by something else | Grey box with a tick | Normal, in italics | The row is in the slice without a state of its own. |
| Contains rows with a state | Box with a solid outline and the lower-left half filled | As for the row's own state | The row has no state of its own, and a row inside it is included or excluded. |

A row is included by something else in three cases. It is inside an included row, the query selects it, or it is a container shown only because a row inside it is in the slice. Select its toggle to give the row a state of its own.

A row with no state of its own that holds an included or excluded row shows the contains mark. The mark shows whether the row is open or closed. It replaces the mark for included by something else, and the row text stays as it was. The filled half is blue when a row inside is included and maroon when one is excluded. When both kinds are inside, the box shows both colours, one on each half. The tooltip reads Contains included rows, Contains excluded rows, or Contains included and excluded rows.

A row that has a state of its own keeps its mark. When a row inside it has the other state, the tooltip adds that, for example Included, contains excluded rows. An included row that holds only included rows adds nothing. A folder counts as a row with a state when all its entities are included or all are excluded.

Select a toggle to move the row to the next state. With the focus on a row, press `i` to include it, `c` to include it with its contents, `x` to exclude it and `u` to clear it. When the row you act on is one of several selected rows, the new state applies to all of them. This lets you include a zone with its contents in one gesture. Use Cmd or Ctrl and click to add a row to the selection, and Shift and click to select a range.

Including a row with its contents does not add channels or flows. A channel or flow appears when its own row is included and its two ends are both in the slice.

An exclusion wins over an inclusion, and a statement in the query cannot remove a row that you included. The slice runs the statements of its query first, then the rows you included, then the rows you excluded.

The toggles change the slice in the same way as the slice editor: the change waits with your other unsaved changes until you click **Save all**. A user who cannot edit the model sees the states, and the toggles do nothing.

The first toggle on a slice that was made before this feature converts the slice without changing what it shows. A slice with no query gets the query `model |> add()`, so it still shows the whole model, and the toggles then add to it or take from it. The **Edit query** field shows the added statement.

The slice editor shows a line such as "Selection: 3 included, 1 excluded" for a slice that has included entries. The line is read-only. Change the entries in the Model Tree.

A slice that shows nothing replaces the diagram with the message "This data slice is empty. Tick elements in the model tree to add them." The Table view still lists the whole model.

### Hide Others and Revert All

In Build, a right-click on a row opens a menu with two actions on the states. They apply to every selected row.

- **Hide others** excludes every other row beside the selected rows, that is, the rows with the same parent. A row that you had included becomes excluded. The selected rows keep their state. The action is available on devices, information nodes, channels, flows, systems and annotations. A zone, a group or a network does not offer it.
- **Revert all** clears the state that you set on the selected rows and on every row inside them. On a row that offers **Hide others**, it also clears the state of every other row with the same parent, which undoes **Hide others**. It does not clear the parent row or the rows inside the other rows. It is available on every row. It is disabled, with the reason as its tooltip, when none of the rows that it would clear has a state of its own.

Both actions change the slice as a toggle does: the change waits with your other unsaved changes until you click **Save all**. A user who cannot edit the model sees Browse, which offers neither. Revert all does not remove a row that the query selects. That row stays in the slice and shows the mark for included by something else.

## Building a Slice from Other Slices

A data slice can inherit the layout of other data slices. The inherited slices give the slice their layout settings and add no items to it. The slice shows what its own query and included rows select, less its excluded rows.

To use the items that the inherited slices show, start a query line from `inherited`. The line `inherited |> add()` takes all of them. A query can mix lines that start from `model` with lines that start from `inherited`. A slice that inherits slices and has no query and no included rows shows the inherited items, as if its query were `inherited |> add()`.

To choose the inherited slices in the slice editor, select **Edit** on the slice. The **Inherit slices** list in the dialog holds the slices this slice inherits:

1. Select **+ Add slice**, then pick a slice from the row's list.
2. Drag a row, or use its handle with the keyboard, to change the order. A later row wins over an earlier row for layout.
3. Clear **Inherit the slices they list too** to use only each listed slice's own items and layout.
4. Select **Save**.

The list offers only slices that can be inherited. It leaves out the slice itself, the transient findings slice, and any slice that would make the slices list each other in a cycle. The Source list of the query builder offers `inherited` only while the list holds a slice.

A model author can also write the slice ids in the `slices` key of the slice, in the model file:

```yaml
dataSlices:
  - id: s-scada
    name: SCADA
    filter:
      program:
        - model |> where("item.tags |> hasAny('scada')") |> add()
  - id: s-comms
    name: Comms
    filter:
      program:
        - model |> where("item.tags |> hasAny('comms')") |> add()
  - id: s-ot-flows
    name: OT flows
    slices: [s-scada, s-comms]
    filter:
      program:
        - inherited |> where("item.kind == 'flow'") |> add()
```

The diagram, the reports and the PNG exports all read a slice that inherits slices the same way. These rules apply to the list:

- An inherited slice that lists slices of its own also contributes those, to any depth, to `inherited` and to layout. Set `flattenSlices: false` on the slice to use only each listed slice's own items and layout.
- Saving refuses slices that list each other in a cycle. An id that names no slice is skipped.
- An inherited slice that has no query and no included rows shows the whole model, so it adds the whole model to `inherited`.
- A layout change saved on an inherited slice carries over to the slice that lists it. A later entry in the list wins over an earlier entry, and the slice's own layout changes win over both.
- A query line that starts from `inherited` in a slice that inherits no slice reports an unsaved register, like any register that was never saved.

## Filter Expression Language (Bonsai)

Filters are written in [bonsai-js](https://github.com/bonsai-js/bonsai-js), a small expression language with pipe-transform syntax.

### Evaluation Context

Each entity is evaluated with two aliases:

```
{ item: entity, [kind]: entity }
```

`kind` is the structural kind of the entity. So `item.tags` and `device.tags` both work on a device. This is the default Tetra context; it differs from Lamina, which uses type-prefixed context only (`cause.`, `event.`, etc.).

The `flow.` and `channel.` aliases work in a program filter, not in a legacy string filter. A `{ program }` filter gives every entity a `kind`, so `flow.tags` and `channel.tags` resolve. A legacy string filter gives only containment-tree nodes a `kind`. A flow or channel there has none, so `flow.tags` resolves to `undefined` and the expression matches nothing. Use `item.` as the alias that works in both.

### `item.kind` And `item.type`

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

### Basic Expressions

```
item.tags |> hasAny("ami")
device.tags |> hasAll("prod", "ami")
item.kind == "network"
item.id |> startsWith("DX-")
item.system == "AMI" && item.tags |> hasAny("ext")
```

The `|>` operator pipes the left-hand value into the right-hand transform as its first argument. All comparison and logical operators (`==`, `!=`, `&&`, `||`, `!`) work normally.

A tag filter matches the tag exactly, including its case. `hasAny("ami")` does not match a tag written `AMI`. To keep tags consistent, use the **Tags** field in the props drawer. As you type, the field lists the tags that other elements in the model already use. Matching ignores case, and tags already on the element are left out. Pick a suggestion to add it. To add a tag that no element uses yet, type it and press Enter.

### Protocol Filters

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

### Multi-Line Filters (AND Semantics)

When a slice's filter has more than one line, the lines are AND-joined at evaluation time. For example, the two lines `item.tags |> hasAny("ami")` and `item.kind == "device"` are equivalent to `item.tags |> hasAny("ami") && item.kind == "device"`. Use multiple lines for readability; use `&&` within a single line to AND conditions that share a prefix.

### OR Semantics

Separate lines are always AND-joined. To match either of two conditions, write them with `||` in a single expression:

```
item.system == "AMI" || item.system == "SCADA"
```

---

## Core Transform Functions

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

### Address-Semantic Transforms

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

## Sentinel Transforms: `cascade()` And `exclude()`

These two transforms act as **end-of-pipe markers** that change how a matched entity is handled, as well as whether it passes.

### Default Behaviour ("Include")

A plain line with no sentinel:

```
item.tags |> hasAny("ami")
```

Matched nodes are kept; unmatched nodes are pruned. Ancestor containers of kept nodes are always retained so the tree stays coherent.

### `cascade()` — Expand Direct Connectors

```
item.tags |> hasAny("ami") |> cascade()
```

When appended to a line, `cascade()` also pulls in every entity directly connected to the matched entities, one hop deep. It reaches them through `nodeA`, `nodeB`, `path` or `devices`.

> **Tetra caveat:** Tetra always pulls in the channels, flows and networks directly connected to a kept node, so `|> cascade()` has no effect in Tetra. In Lamina it selects the chains to expand. The auto-cascade toggle appends it for compatibility with Lamina.

The **auto-cascade toggle** in the data slice edit dialog appends `|> cascade()` to every expression line automatically. The dialog strips the suffix for display and adds it back on save. Lamina's dialog behaves the same way.

### `exclude()` — Subtract After Cascade

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

## The Filter Execution Pipeline

A filter takes a model and returns a narrowed copy. It never changes the original. These rules decide what remains:

- Every node that matches the filter stays, together with its ancestor containers.
- Every channel, flow and network directly connected to a kept node is pulled in.
- A channel or flow with no surviving end is dropped. A channel with several targets survives when at least one target survives.
- An invalid filter returns the full model.

---

## Set-Pipeline Programs: The `follow` Op

The string filter above is a bonsai _predicate_ that selects entities and cascades. The opt-in
`{ program: string[] }` filter shape instead runs an explicit **set-pipeline**.
Each statement reshapes a working
set, and `add()` and `remove()` write it into or out of the result:

```
model |> where("item.kind == \"device\"") |> add()
```

### Tree Node Inclusion — `information` Badges Follow the Same Rule as All Other Nodes

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

### `follow("edge"[, hops])` — Walk One Edge

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

### `includeKinds()` And `excludeKinds()` — Filter by Entity Kind

`includeKinds("a", "b")` keeps the items whose `item.kind` is one of the listed kinds. `excludeKinds("a", "b")` keeps the items whose kind is none of them, including an item with no kind. Both take quoted kind names. `includeKinds()` with no kinds keeps nothing, and `excludeKinds()` with no kinds keeps everything. Neither adds to the result by itself.

### `removeRecursiveDown()` — Subtree Removal

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

### Including All of a Flow's Hop-Nodes

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

## Slice Pruning and Display

A slice always prunes. Tetra removes the entities that the slice does not select, and only the selected subtree renders. A slice saved in shade mode by an earlier version opens pruned, and its saved mode does not mark the slice as modified.

### The Category Rows in the View Menu

The View menu lists four categories and a **Channels** section of three rows. Each row has a `Keep` and a `Hide` button. `Hide` removes that category from the canvas.

### Slice Filter Categories

- **Flows** covers every flow.
- **Networks** removes the network node and its subtree.
- **Annotation Overlay** covers every annotation.
- **Through-Node Runs** is geometry rather than an entity. Removing a run moves no segment index and no hit target.

The **Channels** section has one row for each channel type. The [channel types](/docs/sightline/methodology/tetra-channels-and-flows#channel-types) table says what each type means. Hiding a type removes its channels and leaves the nodes and flows in place.

- **Virtual** removes every virtual channel.
- **Hosting** removes every hosting channel.
- **Network** removes every network channel, whatever its protocol list. A channel with no type counts as a network channel.

A slice saved with the earlier **IP Channels** setting opens with **Network** set to `Hide`. The earlier setting hid only the network channels whose protocol list included `ip` or was empty, so the slice now also hides its serial and other non-IP network channels. Set **Network** to `Keep` and save the view to change this.

### Persistence

The category rows save onto the data slice.

---

## Member Filters on a Zone or Group

Independent of data slice filters, a zone or group can carry two filters of its own. Tetra resolves them when it loads the model. Both fields are in the Advanced section of the zone or group props drawer.

### Member Filter: Prune a Container's Subtree

**Member Filter** takes a bonsai expression, for example `item.tags |> hasAny("ami")`. Only the nodes in that container that match the expression survive. The canvas shows a **funnel badge** (⊻ icon) on the container node to indicate that a subtree filter applies. Hover over the badge to see the expression.

### Include by Expression: Add Members Dynamically

**Include by expression** takes a bonsai expression, for example `item.kind == "device" && item.tags |> hasAll("ami")`. Tetra evaluates it against every entity in the model when the model loads and inserts every match as a member. It suits dynamic groupings that do not need an explicit list of members.

Enter an expression and save to add every matching item to the container. Clear the field and save to remove the rule. Tetra refuses an expression it cannot compile and changes nothing.

The field edits one expression. A container with several such rules shows them in the field, read-only, joined by ` ; `, and keeps all of them when you save other changes. Editing those rules needs the model file, because the drawer offers no control for it.

Tetra does not store the matched items as members, because the expression adds them again on load.
