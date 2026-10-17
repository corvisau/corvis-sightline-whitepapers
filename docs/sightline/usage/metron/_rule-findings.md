A Metron rule can create records from the assessed pairs and attributes of one Tetra model. The Records panel lists the rules, tests an edit and runs them. Records are separate from the findings that a person authors on the [Findings tab](/docs/sightline/usage/metron/findings).

## Which rules run

A run covers one Tetra model. It runs the rules of the rule packages that the requirement packages of that model name in `uses:`. A requirement package counts when an assessment binds it to the model.

```yaml
# requirement package manifest
uses:
  - { kind: rules, id: ot-finding-rules, version: 1 }
```

A rule runs only when its rule package declares `domain: metron` under `meta` and the rule has a program. The package manifest carries the domain, so every rule in the package runs in that domain. A scope rule or an assessment rule has no `emitRecord` step, so a run creates no record from it and leaves it alone.

A model with no assessment runs no rule. An assessment that cannot load adds a warning to the run. When no assessment of the model loads, the run skips each package with a pairs or attributes rule and reports the reason. The existing records stay as they are.

## Write a rule

A rule chooses its rows with `input`.

| Input | One row for | Use it for |
|---|---|---|
| `model` | Each entity of the Tetra model | A record about the model itself |
| `pairs` | Each assessed requirement and entity pair | A record about one outcome |
| `attributes` | Each assessed attribute of a pair | A record about one attribute |

```yaml
rules:
  - ruleId: unmet-attribute
    title: Unmet attribute
    mode: generator
    input: attributes
    program:
      - model |> where("item.attributeState == 'not-met'") |> emitRecord("{ requirementId: item.requirementId, entityId: item.entityId, attributeName: item.attributeName }", "{ state: item.attributeState }", "[item.entityId, { scope: 'attribute', id: item.id }]")
```

### Fields of a pair row

| Field | Content |
|---|---|
| `id` | The id of the pair row |
| `packageId`, `packageVersion` | The requirement package of the assessment |
| `packageScope` | `package:` followed by the package id, for a reference to the requirement |
| `modelId` | The id of the Tetra model |
| `requirementId`, `requirementVersion` | The requirement and its live version |
| `entityId`, `entityKind`, `entityVersion` | The entity and its live version |
| `state`, `fundamentalState` | The assessment state of the pair |
| `outcome`, `fundamentalOutcome` | The outcome of the pair, and its fundamental outcome |
| `requiresRevalidation` | Whether a version moved since the assessor signed the pair off |
| `assessedRequirementVersion`, `assessedEntityVersion` | The versions the assessor assessed |
| `assessedBy`, `assessedAt` | The assessor and the time |
| `attributeStates` | A map from attribute name to `met`, `partial` or `not-met` |

### Fields of an attribute row

An attribute row holds every field of its pair row. It adds these:

| Field | Content |
|---|---|
| `attributeName` | The name of the attribute |
| `attributeState` | `met`, `partial` or `not-met` |
| `pairId` | The id of the pair row that owns the attribute |

An attribute row exists only for an attribute that the assessor has assessed.

### Links

A record stores what it depends on as links. A rule declares the links in `emitRecord`. A link names an id, and a `scope` when the id is not an entity of the model.

| Scope | Id | What the record stores |
|---|---|---|
| none | An entity of the Tetra model | The version of the entity |
| `package:` and the package id | A requirement id | The version of the requirement |
| `pair` | The id of a pair row | A hash of the outcome and the attribute states |
| `attribute` | The id of an attribute row | A hash of that attribute state |

A change to the notes or the evidence of a pair does not move a `pair` or `attribute` link.

## Open the panel

Right-click an assessment in the Models tree and select **Metron: Open Records**. The panel opens for the Tetra model of the assessment. The command **Metron: Open Records** in the palette asks which assessment.

Each model has one panel, and a second request reveals the open one. Two assessments of the same model open the same panel. The browser version of Metron does not show this panel.

## The Rules tab and the Records tab

The panel has the two tabs and the controls of the Tetra Rules panel, including **Columns**, **Filter** and **Group** on both tabs. See [Rules and records](/docs/sightline/usage/tetra/rules-and-findings) for the drawer, **Test rule**, **Save** and **Run rules**. The records list is read-only; accept a finding in the assessment's **Findings** tab. Two differences apply in Metron:

- The tab lists the rules of the rule packages that the requirement packages name. The empty message says that no rule package is named.
- The query builder offers the standard functions without `emit`. A generator rule creates records, and `emit` tags an entity for an assessment.

A rule save changes only that rule in its package file. Other models and requirement packages that use the package see the change.

## Run rules

**Run rules** shows how many records a run creates and how many it closes. **Confirm** writes them. A run never deletes a record and never changes the status, severity or description that a person set.

A run covers each rule package once for the model. A package with a program error is skipped, and its records stay unchanged.

### Manual records

A run also refreshes the links of manual records. It resolves each link again. When a version or a hash moved, the run records the new value and the record shows **Source changed**. The data and the set of links stay as the person set them.

A link under a scope that the run does not hold stays as stored, and the run reports a warning. A link to an id that no longer exists shows as missing, which counts as a change.

## Licence

A run and a rule save need an edit licence for Metron. A refused action shows its reason in the panel. See [Licensing](/docs/sightline/usage/shared/licensing).
