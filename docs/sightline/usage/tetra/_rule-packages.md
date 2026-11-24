A Tetra model produces records from the rule packages it uses. A record is an entry that a rule creates for one relation and one zone it enters. A model that uses no rule package produces no records.

Records are separate from the Metron findings that the [Findings overlay](/docs/sightline/usage/tetra/findings-overlay) draws on the canvas.

## Choose the Packages a Model Uses

Open the **Packages** tab of the Rules panel. There, add a rule package to the model, pin its version or remove it. See [Rules and records](/docs/sightline/usage/tetra/rules-and-findings#the-packages-tab).

The Metron editor of a rule package lists every Tetra model that uses it. See [Tetra models, records, problems and cause links](/docs/sightline/usage/metron/component-packages#tetra-models-records-problems-and-cause-links).

Tetra finds a package by its id among the rule packages installed under the workspace root. The location of the package does not matter. When a version is not pinned, the model uses the single installed version of the package.

## Create a Rule Package

Create a rule package in the Models tree and edit it in the rule package editor. See [Component packages](/docs/sightline/usage/metron/component-packages). Set the package **Domain** to `tetra`. Tetra runs only the packages with that domain, and every rule in a package runs in its domain. Tetra reports a package without a domain in the Problems panel and runs none of its rules. The **Domain** select is on the **Package meta** tab.

Select **+ Add rule** in the editor to create a rule. The drawer asks for the fields that [Rules and records](/docs/sightline/usage/tetra/rules-and-findings#edit-a-rule) describes.

### Example: A Relation Enters a Zone

This rule creates a record for each zone that a cross-zone channel or flow enters. Set these fields in the drawer.

| Field | Value |
|---|---|
| Rule id | `cross-zone-relation` |
| Title | Relation enters a zone |
| Description | A channel or flow crosses a zone boundary and enters a zone. |
| Mode | `generator` |
| Input | `model` |
| Hide from consumer lists | Off |
| Data fields | `type`, with the type `text` |

Select **Edit query** and enter these two statements.

```
model |> where("item.kind |> oneOf('channel', 'flow') && item.isCrossZone && (item.kind == 'flow' ? item.infoDirection : item.direction) != 'b-to-a'") |> emitRecord("{ zoneId: item.zoneB, relationId: item.id }", "{ type: item.type }", "[item.id, item.zoneB]")
model |> where("item.kind |> oneOf('channel', 'flow') && item.isCrossZone && (item.kind == 'flow' ? item.infoDirection : item.direction) != 'a-to-b'") |> emitRecord("{ zoneId: item.zoneA, relationId: item.id }", "{ type: item.type }", "[item.id, item.zoneA]")
```

The `direction` of a channel and the `infoDirection` of a flow decide the zone. A relation with no direction, or a bidirectional one, enters both zones and creates two records.

Key a Tetra record on `zoneId`. The Lamina Findings view places a Tetra record in a zone by the `zoneId` part of its key. A record with no `zoneId` part, or with one outside the zones in the scope of the Lamina model, does not appear in that view. A note above the tree gives the count of such records.

Two packages that define the same rule id are two rules, and each keeps its own records.

### Example: A Flow That a Lamina Cause Can Link

A Lamina cause that links a Tetra record takes its zones from the data of that record. The cause is Tetra-linked only when the data holds all three of these fields. See [Tetra-linked causes](/docs/sightline/usage/lamina/finding-links#tetra-linked-causes).

| Data field | Holds | Value in the rule |
|---|---|---|
| `sourceZone` | The zone the flow starts in | `item.zoneA` |
| `targetZone` | The zone the flow ends in | `item.zoneB` |
| `pathZones` | The zones the flow passes through, from the source zone to the target zone | `item.pathZones` |

A record without all three fields still links to the cause. The cause then takes no zones from it, and its drawer reads "No source zone".

This rule creates one record for each flow between two zones. Set these fields in the drawer.

| Field | Value |
|---|---|
| Rule id | `tetra-linked-flow` |
| Title | Flow that a Lamina cause can link |
| Description | A flow between two zones, with the zones a Lamina cause needs. |
| Mode | `generator` |
| Input | `model` |
| Hide from consumer lists | Off |

Select **Edit query** and enter this statement. Select **Apply**, and the drawer takes `flowId` as the key part and the three fields as data fields.

```
model |> where("item.kind == 'flow' && item.isCrossZone && item.infoDirection != 'b-to-a'") |> emitRecord("{ flowId: item.id }", "{ sourceZone: item.zoneA, targetZone: item.zoneB, pathZones: item.pathZones }", "[item.id]")
```

`item.isCrossZone` keeps only the flows with a zone at both ends. `item.pathZones` always runs from the zone of `nodeA` to the zone of `nodeB`, so the rule leaves out a flow with the `infoDirection` `b-to-a`. A flow with no `infoDirection`, or a bidirectional one, starts at `nodeA`. To also give a cause a flow that runs from `nodeB` to `nodeA`, add a second statement that keeps `item.infoDirection == 'b-to-a'` and emits `sourceZone: item.zoneB, targetZone: item.zoneA, pathZones: item.pathZones |> reverse`.

#### Update a Flow Rule Written for an Earlier Release

A flow rule written for an earlier release can emit `sourceZone` alone. A cause that links one of its records is not Tetra-linked. To fix it:

1. Add `targetZone: item.zoneB` and `pathZones: item.pathZones` to the data of the `emitRecord` step.
2. Select **Apply**, so that **Data fields** lists the two new fields.
3. Save the rule package.
4. Select **Run rules** on the **Records** tab of the Rules panel.
5. In Lamina, select **Accept** on the record link of each cause that links the rule. A cause keeps the data of its last Accept, so the new fields reach it only then.

### Chain Flows in Series

An attack path often pivots through an intermediate device. A remote-access flow reaches an engineering workstation, and an engineering flow from there reaches a safety system. A Lamina cause links one record, so a rule derives the whole path as one record with the `chains` step.

`chains(3)` replaces the flows it receives with one row for each chain of flows in series. A chain hop is one flow of the chain. A pivot is a device where one chain hop ends and the next starts. The number is the hop limit, the most flows in one chain. Without a number the limit is 3.

Select **Edit query** and enter this statement. Replace `safety` with the id of the zone that the chain must enter.

```
model |> where("item.kind == 'flow'") |> chains(3) |> where("item.isCrossZone && item.zoneB == 'safety'") |> emitRecord("{ chain: item.id }", "{ sourceZone: item.zoneA, targetZone: item.zoneB, pathZones: item.pathZones, pivots: item.pivots, pivotZones: item.pivotZones }", "item.flows")
```

The statement writes `pivots` and `pivotZones` into the data of each record. A Lamina cause names the pivots in its label and shows each pivot on the box of its zone. A rule that leaves them out still gives the cause its zones.

A chain row has these fields.

| Field | Holds |
|---|---|
| `id` | The ordered flow ids joined with `>`. A chain of one flow has the id of that flow |
| `flows` | The flow ids, in order |
| `pivots` | The ids of the pivot devices, in order |
| `pivotZones` | The zone of each pivot, in the same order. A pivot in no zone has no zone |
| `nodeA`, `nodeB` | The device at the start and the device at the end of the chain |
| `zoneA`, `zoneB` | The zones of those two devices |
| `pathZones` | The zones of every chain hop, from `zoneA` to `zoneB`. The zone of a pivot is listed once |
| `hops` | The number of flows in the chain |
| `isCrossZone` | `true` when `zoneA` and `zoneB` are both set and differ |

The step follows these rules.

- The step uses only the flows that the statement passes in. Filter before `chains` to choose the chain hops, and after it to keep the chains.
- A chain hop runs from `nodeA` to `nodeB`. A `b-to-a` flow runs from `nodeB` to `nodeA`, and a bidirectional flow runs either way. `pathZones` follows the direction of the chain.
- A chain never uses a device twice.
- A flow with a list in `nodeB` gives one chain for each target. A bidirectional flow gives one chain for each direction. The id of such a chain ends with `@`, the start device, `>` and the end device.
- A flow on its own is a chain of one flow. A rule that also emits single flows, as in the previous example, records those flows twice.
- More than 1000 chains stop the statement with an error. Lower the hop limit or pass fewer flows.

The key of the record is the chain id. A new flow between two others gives a new chain id and so a new record, and the earlier record reads Closed.

### Read the Records of Other Rules

A rule with the input `records` reads the records of the rules in **Depends on**. It needs at least one entry. The following rule creates one record for each zone that more than three relations enter. Set **Mode** to `generator`, **Input** to `records`, and **Depends on** to `cross-zone-relation`. Then enter this statement in the query builder.

```
model |> where("item.ruleId == 'cross-zone-relation' && !item.closed") |> groupBy("item.key.zoneId") |> where("item.count > 3") |> emitRecord("{ zoneId: item.id }", "{ count: item.count }", "item.members |> map(.id)")
```

Write a rule of another package in **Depends on** as `package-id/rule-id`. See [Rules and records](/docs/sightline/usage/tetra/rules-and-findings) for the fields of a row.

Edit a rule in the Rules panel of Tetra. The Metron rule package editor opens the same package in the same rule drawer. It keeps **Mode**, **Hide from consumer lists** and **Data fields** when it saves.

The rule drawer opens 720 pixels wide. Drag its left edge to resize it, and double-click the edge to return to 720 pixels. The drawer keeps the width you set for the next rule you open.

## Write a Diagnostic Rule

A diagnostic states a fact about the current model. Tetra shows each open diagnostic in the Problems view. A diagnostic is a record of a rule that hides its records from the consumer lists. It has a key, a journal and a closed state like any other record.

Create a `generator` rule with **Hide from consumer lists** switched on that reads the model. The program ends in `emitRecord` with the entity as the key, the message and level as data, and the entity as the only link.

Enter this statement for a rule that flags each zone that a cross-zone relation enters or leaves.

```
model |> where("item.kind == 'zone' && item.isCrossZoneExposed") |> emitRecord("{ entityId: item.id }", "{ message: 'Zone ' + item.id + ' is entered or left by a cross-zone relation', severity: 'warning', entityKind: item.kind }", "[item.id]")
```

| Part | Value |
|---|---|
| Key | `{ entityId: item.id }`. The entity id is the key, so a rule gives one diagnostic for each entity. |
| `message` | The text of the Problem. Tetra shows it after the rule id and a colon. A record with no text message is not a diagnostic and is not shown. |
| `severity` | `error`, `warning` or `info`. Any other value, or none, gives `warning`. |
| `entityKind` | The kind of the entity, such as `channel`. Tetra uses it to find the file and line of the entity. |
| Link | `[item.id]`. The link pins the version of the entity, so an edit to the entity moves the pin. |

The row the program pipes into `emitRecord` is the entity that the diagnostic is about. The rule must not depend on a rule that is not hidden in the same package. Tetra reports a Problem that names the rule, and the package shows no diagnostic.

The Rules panel edits a diagnostic rule and tests it. See [Make a rule a diagnostic rule](/docs/sightline/usage/tetra/rules-and-findings#make-a-rule-a-diagnostic-rule).

### Rule Package Diagnostic Display

Tetra computes the diagnostics in memory each time the model loads. Opening a model writes no file. A save and a change to a model file load the model again, so the Problems view follows them. A change that is not yet saved does not change a diagnostic.

A save writes the diagnostics that changed to the register of the package, under `findings/<package id>/`, and adds one entry to its journal. A save that changes no diagnostic writes nothing.

- A new diagnostic adds a `created` entry.
- An edit to the entity that the diagnostic is about adds a `changed` entry, because the link pin moves. The message can stay the same.
- A fix adds a `closed` entry. The record stays in the register, closed, and the Problems view no longer shows it.

The Problems view shows each diagnostic on the file and line that declare its entity. When Tetra cannot find the entity, the Problems view shows it on the model manifest. The browser version of Tetra lists it in the Problems list with no file.

A rule that fails to run adds one Problem that names the rule and the cause. The diagnostics of the other rules still appear.

## Use the Starter Package

The Tetra starter package has the id `tetra-rules` and version 2. It holds three rules.

| Rule | Mode | Reports |
|---|---|---|
| `cross-zone-relation` | `generator` | A record for each zone that a cross-zone channel or flow enters. |
| `path-hop-missing` | `generator`, hidden | A channel or flow whose path names a hop that the model does not define. |
| `path-via-missing` | `generator`, hidden | A channel or flow whose path names a `via` channel that the model does not define. |

Each path rule gives one Problem for each channel or flow. The Problem lists every missing id.

A model that does not use the package gets the same warnings from the loader. A model that uses the package gets each warning once, from the rule. If a model uses only one of the two path rules, the loader keeps the warnings of the other.

A model that uses version 1 of the package has `cross-zone-relation` only. To add the path rules, add each one as a new rule in the rule package editor. Then set the version to 2 on the **Package meta** tab. Set **Mode** to `generator` and switch on **Hide from consumer lists**. Enter these two statements for `path-hop-missing`, with the title Path hop does not exist.

```
model |> where("item.kind != 'channel' && item.kind != 'flow'") |> save("known")
model |> where("item.kind |> oneOf('channel', 'flow') && (item.path |> missingRefs(known) |> count) > 0") |> emitRecord("{ entityId: item.id }", "{ message: (item.kind == 'channel' ? 'Channel ' : 'Flow ') + item.id + ' routes through path hops that do not exist in the model: ' + (item.path |> missingRefs(known) |> join(', ')) + ' - each hop is skipped and the connection routes through its remaining hops.', severity: 'warning', entityKind: item.kind }", "[item.id]")
```

Create `path-via-missing` the same way, with `item.kind == 'channel'` in the first statement and `item.pathVias` in place of `item.path`.

The first statement saves every row that is not a channel or a flow into the `known` register. The second statement keeps each channel or flow with a hop that is not in `known` and creates one diagnostic record for it. `missingRefs(register)` returns the ids of a value that the register does not hold.

A rule sees a row of kind `system` for each system of the model. Each channel and flow row carries these fields for its path:

| Field | Holds |
|---|---|
| `pathVias` | The `via` ids of the path that are not patterns |
| `pathZones` | The zones the path passes through, in path order, from `zoneA` to `zoneB`. A hop in no zone, and a zone that repeats the one before it, are left out |
| `pathDevices` | The ids of the devices among the path hops, in path order |

A hop that is a pattern, or an id the model does not define, adds nothing to either list. A rule can write `pathZones` or `pathDevices` into the data of a record with `emitRecord`. A record that a Lamina cause links needs `pathZones` in its data. See [Example: a flow that a Lamina cause can link](#example-a-flow-that-a-lamina-cause-can-link).

`pathZones` always runs from `zoneA` to `zoneB`, whatever the direction of the flow. For a `b-to-a` flow, write `item.pathZones |> reverse` to give the zones from the source to the target. `reverse` returns a new list, so `item.pathZones` keeps its order for the rest of the rule.

## Run the Rules

A run reads the model and runs every rule of the packages that the model uses. Select **Run rules** on the **Records** tab of the Rules panel to start one. It writes the results to a register under `findings/<package id>/` in the workspace root. A second run with no change to the model changes nothing.

The run is the `finding-runs` operation of the storage server. See [Model routes](/docs/sightline/reference/api/models) in the API reference. A run answers a diagnostic for each package entry that matches no package, or more than one. Pin a version on the entry that matches more than one.

A real run saves the problems it reports for each package. The Metron editor of the package shows them in its **problems** column until the next run. A dry run saves nothing.

A stored record stays readable after its package leaves the model. The rules behind it are then unavailable, so a run no longer updates it.
