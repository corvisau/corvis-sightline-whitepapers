A Tetra model produces records from the rule packages it lists in its manifest. A record is an entry that a rule creates for one relation and one zone it enters. A model that lists no rule package produces no records.

Records are separate from the Metron findings that the [Findings overlay](/docs/sightline/usage/tetra/findings-overlay) draws on the canvas.

## List the packages a model uses

Add a `uses:` key to the model manifest. Each entry names one rule package by its id.

```yaml
uses:
  - { kind: rules, id: tetra-rules, version: 2 }

blockdiagram:
  meta:
    id: basic
    name: Basic
```

| Field | Value |
|---|---|
| `kind` | `rules`. No other kind is valid here. |
| `id` | The `meta.id` of the rule package. |
| `version` | The `meta.version` of the package. Leave it out to use the single installed package with that id. |

Tetra finds the package by id among the `*.rules.metron.yaml` files under the workspace root. The path of the package does not matter.

The **Packages** tab of the Rules panel edits this list without a text editor, and keeps the comments of the manifest. See [Rules and records](/docs/sightline/usage/tetra/rules-and-findings#the-packages-tab).

A `uses:` list that does not match this shape produces an error in the Problems panel. The model still opens, and Tetra reads the list as empty.

## Write a rule package

A rule package is a manifest and one or more rule files.

```yaml
# tetra-rules.rules.metron.yaml
kind: rules
meta:
  id: tetra-rules
  version: 2
  name: Tetra starter rules
  domain: tetra
files:
  - rules/**/*.rules.yaml
```

`meta.domain` names the app that runs the rules of the package. Tetra runs a package with `domain: tetra`. Metron runs a package with `domain: metron`. A package has one domain, and every rule in it runs in that domain. A manifest with no `meta.domain` does not load. Tetra reports the package in the Problems panel and runs none of its rules.

The manifest finds rule files through `files:`, which is required. It lists file paths or glob patterns relative to the manifest. Each file holds a `rules:` list.

```yaml
# rules/cross-zone-relation.rules.yaml
rules:
  - ruleId: cross-zone-relation
    title: Relation enters a zone
    description: A channel or flow crosses a zone boundary and enters a zone.
    mode: generator
    input: model
    hide: false
    dataFields:
      - { name: type, type: text }
    program:
      - model |> where("item.kind |> oneOf('channel', 'flow') && item.isCrossZone && item.direction != 'b-to-a'") |> emitRecord("{ zoneId: item.zoneB, relationId: item.id }", "{ type: item.type }", "[item.id, item.zoneB]")
      - model |> where("item.kind |> oneOf('channel', 'flow') && item.isCrossZone && item.direction != 'a-to-b'") |> emitRecord("{ zoneId: item.zoneA, relationId: item.id }", "{ type: item.type }", "[item.id, item.zoneA]")
```

This rule creates a record for each zone that a cross-zone channel or flow enters. The `direction` of the relation decides the zone. A relation with no direction, or a bidirectional one, enters both zones and creates two records.

Two packages that define the same `ruleId` are two rules, and each keeps its own records.

### Read the records of other rules

A rule with `input: records` reads the records of the rules that it names in `dependsOn`. It needs at least one entry. The following rule creates one record for each zone that more than three relations enter.

```yaml
rules:
  - ruleId: zone-many-relations
    title: Zone is entered by several relations
    mode: generator
    input: records
    dependsOn: [cross-zone-relation]
    hide: false
    program:
      - model |> where("item.ruleId == 'cross-zone-relation' && !item.closed") |> groupBy("item.key.zoneId") |> where("item.count > 3") |> emitRecord("{ zoneId: item.id }", "{ count: item.count }", "item.members |> map(.id)")
```

Write a rule of another package as `package-id/rule-id`. See [Rules and records](/docs/sightline/usage/tetra/rules-and-findings) for the fields of a row.

Edit a rule in the Rules panel of Tetra, or edit the package as a text file. See [Rules and records](/docs/sightline/usage/tetra/rules-and-findings). The Metron rule editor opens the same package in the same rule drawer and keeps `mode`, `hide` and `dataFields` when it saves.

## Write a diagnostic rule

A diagnostic states a fact about the current model. Tetra shows each open diagnostic in the Problems view. A diagnostic is a record of a rule that hides its records from the consumer lists, so it has a key, a journal and a closed state like any other record.

Write a `generator` rule with `hide: true` that reads the model. The program ends in `emitRecord` with the entity as the key, the message and level as data, and the entity as the only link.

```yaml
rules:
  - ruleId: zone-exposed
    title: Zone is entered or left by a cross-zone relation
    mode: generator
    hide: true
    program:
      - model |> where("item.kind == 'zone' && item.isCrossZoneExposed") |> emitRecord("{ entityId: item.id }", "{ message: 'Zone ' + item.id + ' is entered or left by a cross-zone relation', severity: 'warning', entityKind: item.kind }", "[item.id]")
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

### Where a diagnostic appears

Tetra computes the diagnostics in memory each time the model loads. Opening a model writes no file. A save and a change to a model file load the model again, so the Problems view follows them. A change that is not yet saved does not change a diagnostic.

A save writes the diagnostics that changed to the register of the package, under `findings/<package id>/`, and adds one entry to its journal. A save that changes no diagnostic writes nothing.

- A new diagnostic adds a `created` entry.
- An edit to the entity that the diagnostic is about adds a `changed` entry, because the link pin moves. The message can stay the same.
- A fix adds a `closed` entry. The record stays in the register, closed, and the Problems view no longer shows it.

The Problems view shows each diagnostic on the file and line that declare its entity. When Tetra cannot find the entity, the Problems view shows it on the model manifest. The browser version of Tetra lists it in the Problems list with no file.

A rule that fails to run adds one Problem that names the rule and the cause. The diagnostics of the other rules still appear.

## Use the starter package

The Tetra starter package has the id `tetra-rules` and version 2. It holds three rules.

| Rule | Mode | Reports |
|---|---|---|
| `cross-zone-relation` | `generator` | A record for each zone that a cross-zone channel or flow enters. |
| `path-hop-missing` | `generator`, hidden | A channel or flow whose `path` names a hop that the model does not define. |
| `path-via-missing` | `generator`, hidden | A channel or flow whose `path` names a `via` channel that the model does not define. |

Each path rule gives one Problem for each channel or flow. The Problem lists every missing id. A `via` that contains `*` is a pattern, and the rules do not check it.

A model that does not list the package gets the same warnings from the loader. A model that lists the package gets each warning once, from the rule. If a model lists only one of the two path rules, the loader keeps the warnings of the other.

A model that holds version 1 of the package has `cross-zone-relation` only. To add the path rules, create `rules/path-hop-missing.rules.yaml` in the package and set `meta.version` to 2 in the package manifest. The following file is `path-hop-missing`. Write `path-via-missing` the same way, with `item.kind == 'channel'` in the first statement and `item.pathVias` in place of `item.path`.

```yaml
rules:
  - ruleId: path-hop-missing
    title: Path hop does not exist
    mode: generator
    hide: true
    program:
      - |-
        model |> where("item.kind != 'channel' && item.kind != 'flow'") |> save("known")
      - |-
        model |> where("item.kind |> oneOf('channel', 'flow') && (item.path |> missingRefs(known) |> count) > 0") |> emitRecord("{ entityId: item.id }", "{ message: (item.kind == 'channel' ? 'Channel ' : 'Flow ') + item.id + ' routes through path hops that do not exist in the model: ' + (item.path |> missingRefs(known) |> join(', ')) + ' - each hop is skipped and the connection routes through its remaining hops.', severity: 'warning', entityKind: item.kind }", "[item.id]")
```

The first statement saves every row that is not a channel or a flow into the `known` register. The second statement keeps each channel or flow with a hop that is not in `known` and creates one diagnostic record for it. `missingRefs(register)` returns the ids of a value that the register does not hold.

A rule sees a row of kind `system` for each system of the model. Each channel and flow row carries `pathVias`, the `via` ids of its `path` that are not patterns.

## Run the rules

A run reads the model and runs every rule of the packages in `uses:`. Select **Run rules** on the **Records** tab of the Rules panel to start one. It writes the results to a register under `findings/<package id>/` in the workspace root. A second run with no change to the model changes nothing.

The run is the `finding-runs` operation of the storage server. See [Model routes](/docs/sightline/reference/api/models) in the API reference. A run answers a diagnostic for each `uses:` entry that matches no package, or more than one. Add a `version` to the entry that matches more than one.

A stored record stays readable after its package leaves `uses:`. The rules behind it are then unavailable, so a run no longer updates it.
