This page describes how Lamina, Tetra and Metron work together. It covers how the apps find each other's models, what each app may write, and the workflows that cross an app boundary.

## The apps

| App | Models | Files |
|---|---|---|
| Tetra | Architecture models: zones, devices, networks, channels, flows, information and data slices | `*.tetra.yaml` |
| Lamina | Risk models: causes, exposures, controls, consequences and the risk matrix | `*.bowtie.yaml` |
| Metron | Requirement packages, assessments, rules and findings | `*.metron.yaml`, `*.req.yaml`, `*.assessment.yaml`, `*.rules.yaml`, `*.rule-findings.jsonl` |
| Sightline | Coordinator: one activity-bar entry with a combined model tree over Tetra, Lamina and Metron | — |

Lamina, Tetra and Metron run inside the Sightline extension. The apps share the model files in the workspace.

### Shared authoring language

Lamina and Tetra resolve `template:` inheritance and Handlebars partials with the same engine. A modeller who moves between the apps meets one authoring language. Both apps also ship the same **Template Editor** and the same **Constants Editor**. See [Tetra templates](/docs/sightline/usage/tetra/templates).

## How the apps find each other

Two rules hold for every cross-app interaction.

1. **An app finds another app's files by scanning the workspace for well-known file patterns.** No setting lists another app's files. A new file is found the next time the app loads.
2. **Each app writes only its own model files.** An app that owns a link writes that link's reference field in its own record only. Lamina never writes a record or a finding, and Metron never writes a Lamina cause or control.

A cross-app link is a plain string reference. The side that owns the lifecycle of the relationship stores it. The other side resolves it when it reads, as a computed back-reference that is never saved. This design has two effects:

- A reference is never stored on both sides, so the two sides cannot drift apart.
- A reference that no longer resolves shows as "—". It does not stop the model from loading.

### Model identity

Every model has a stable id that does not depend on its file path:

- A Tetra model has `meta.id`.
- A Lamina model has `meta.id`. Lamina adds it when a model has none.
- A Metron package has `meta.id` and `meta.version`.
- A Metron assessment has a top-level `id`.

Ids are unique within each kind of model. A Lamina id and a Tetra id can share a string without a conflict. Every Metron package kind also compares on its version. A Metron rule package can share an id with a requirement package. Metron tells the package kinds apart by the `kind` field of each manifest. Every Metron outcome record and finding reference carries the model id, so two models that share an entity id never collide.

### Renaming, moving and deleting a model

A Tetra or Lamina model can be renamed, moved to another folder and deleted. Rename changes only the name that the model is listed under. Move re-files the model's directory and changes no id, so every reference to the model keeps resolving: a Metron assessment finds the model from `meta.modelId`, and a Lamina model finds it from `tetraModel`.

Delete is refused while another file still names the model's id. The refusal lists each referrer, and nothing is removed. These files count as referrers:

- A findings register for the model.
- A Metron assessment that binds the model in `meta.modelId`.
- A Lamina manifest that scopes to the model in `tetraModel`.

Move and delete act on the whole directory of a model, so both are refused for a directory that holds another model, and for the workspace root. A move is also refused when the target directory exists, or when the model reads files outside its own directory.

A folder can be renamed, moved and deleted as well. Rename and move change no id, so no reference changes. Delete is refused while the folder holds any file, so a folder is removed only when it is empty. Rename and move are refused for the `findings` folder and everything under it, for a folder inside a model's directory, and for a folder that holds a Metron package. A move is also refused when a model in the folder reads files outside it, and when the destination exists.

## Workflow: compliance assessment (Metron and Tetra)

Metron loads a requirement package and binds it to a Tetra model through an assessment file. The assessment file names the Tetra model by its id, in `meta.modelId`. Metron finds the model from that id each time it loads the assessment, so the model can move to another folder without an edit to the assessment. Metron then resolves each pair of one requirement and one entity.

A reference from one model to another holds an id and never a folder or a path.

Each pair carries its outcome, evidence and notes in the assessment file. A run of the assessment also keeps an assessment register, a `.rule-findings.jsonl` file with one record for each requirement and in-scope entity. Each record also holds its pair's outcome, state and evidence, and every change to the assessment keeps the two in step. The records are not shown; the assessment file stays the source of the outcomes. A pair has one of three built-in states:

| State | Meaning |
|---|---|
| `draft` | The pair is editable. An automatic outcome from a rule starts as `draft`, so a machine suggestion never counts as a human sign-off. |
| `in-progress` | The pair is being worked and is editable. |
| `done` | The pair is read-only until you reopen it. Reopening adds an entry to the log of the assessment file. |

Metron does not log outcome edits. Version control records those.

Suppose the requirement or the entity changes after a pair is `done`. Metron then sets the `requiresRevalidation` flag on the pair. The flag shows as the **Reval** column of the worklist. The pair stays `done` and still counts toward coverage. Only a person who reopens the pair clears the flag.

The worklist narrows in two independent ways, and they combine with AND:

- A compliance slice, which is a named filter over categories, tags, target kinds and entity scope. It never changes what Metron assesses. The requirement's scope rule controls that.
- A filter on the worklist view. See [Worklist filtering](/docs/sightline/usage/metron/worklist-filtering).

### Coverage overlay in Tetra

Tetra can draw the assessment coverage of each entity on its diagram. The overlay is optional. Tetra computes the figures with Metron's rules from every assessment that names the Tetra model. The figures therefore equal the worklist and the Portfolio Roll-Up, and nothing is saved. When the licence does not name Metron data, the overlay shows a notice that gives the reason. See [Coverage overlay](/docs/sightline/usage/tetra/coverage-overlay).

### How Tetra reads Metron data

Tetra reads the Metron findings and coverage of the model that it has open from the assessments in the workspace. These reads need the `view:metron` licence grant, and a refused read reports its reason. A rule package is read with the grant of its domain: Tetra's own rule packages, and the findings their rules produce, need only the Tetra grant. The Metron findings that Tetra reads for its badges stay on `view:metron`. See [Licensing](/docs/sightline/usage/shared/licensing).

The storage server exposes the same findings and coverage, and the causes that cover a Tetra entity, as routes. See the [API reference](/docs/sightline/reference/api/overview).

### How Lamina reads Tetra records

Lamina reads the records that the rule packages of a Tetra model create. It uses them for the state of each record that a cause links, and for the records in the declared scope that no cause links. It also lists the Tetra models under the Lamina model's folder, which the Model Properties drawer offers as the choices for `tetraModel` and `tetraZones`.

The VS Code host reads these answers through the storage server's routes, in process, by the Lamina model's own id. A manifest outside the first workspace folder, or one that shares its `meta.id` with another manifest, has no id to read by, so the host shows no record data for it. A Lamina app serves four routes, and each takes the Lamina model's own id:

| Route | Answer |
|---|---|
| `GET /cross-app/models/:modelId/tetra-scope` | The Tetra models of the workspace, each with the zone ids it declares |
| `GET /cross-app/models/:modelId/finding-links` | The drift of every record that a cause of the model links: one of eight states, including `unreadable` for a register that cannot be read |
| `GET /cross-app/models/:modelId/causes/:causeId/finding-links` | The same entries for one cause |
| `GET /cross-app/models/:modelId/finding-inbox` | The records in the model's scope, each marked linked or not |

The `tetra-scope` read needs no Metron licence grant. The three record reads follow the server's read check, as every finding read does. See the [Discovery routes](/docs/sightline/reference/api/discovery).

## Workflow: rule, finding, cause (Metron, Tetra and Lamina)

This workflow links a compliance gap to the risk register.

```
Metron: assessed pairs of requirement and entity
        |
        v  promote
Metron: manual finding in the register of the Tetra model
        (rule package assessment-findings, rule assessment-gap)
        status: open, reviewed or resolved (you set it)
        links: the versions of the entity and the requirement
        |
        v  Lamina: the findings inbox lists Metron findings that no cause links
Lamina: Create cause from finding
        |  creates a draft cause whose vector holds the record link
        |  pinned at the state of the finding that it accepted
        v
Lamina: the cause shows the state of its linked finding
```

A Metron user promotes assessed pairs into a **finding**. A Lamina user then creates a **cause** that links the finding.

- **A finding is a record that you edit.** Its status is `open`, `reviewed` or `resolved`, and you set it. A run of the model's Metron rules refreshes its links, and the finding shows **Source changed** when a version moved.
- **The cause stores the one persisted link.** The link is in the cause's vector, with `scope: record`, and the accepted pin sits on the link. The link names the finding by id and by the Tetra model that holds it. A finding stores nothing about a cause. Lamina computes where a finding is used from the links of its models.
- **Metron accepts a finding; Tetra only reads it.** **Accept** in Metron's **Findings** tab pins the links of the finding and records the acceptance in the finding's register. Tetra's **Records** tab lists the same record and changes none.
- **Lamina writes only its own model.** Creating the cause and accepting a change commit the cause, with the model's other staged changes, on **Save all**. Metron's finding routes are not involved.
- **Scope and assessment rules do not create findings.** The **Rules** tab is for authoring. A scope rule or an assessment rule that a requirement names narrows the scope or suggests an outcome. A rule in a package with `meta.domain: metron` creates records instead. See [Records from assessed pairs](#workflow-records-from-assessed-pairs-metron).

### Findings on the Tetra diagram

Tetra draws a findings badge on zones, groups, devices, networks and information nodes. Hover over the badge to see the findings for that entity. The hover card offers three actions:

- **Open in Table** opens the Tabular Editor, filtered to the entity.
- **Open in Drawer** opens the props drawer of the entity, with a read-only Findings section.
- **Jump to Metron** opens the assessment that is bound to the finding, on the **Findings** tab, with that finding selected. When a package has more than one version, Metron uses the newest.

A second jump for the same assessment reuses the open tab and moves the selection to the new finding. Jump to Metron finds the assessment from the model id and package id that the finding stores. See [Findings overlay](/docs/sightline/usage/tetra/findings-overlay).

## Workflow: records from assessed pairs (Metron)

A Metron rule in a rule package with `meta.domain: metron` creates records from the assessed pairs and attributes of one Tetra model. Tetra and Metron parse a rule with one schema, so a package of either domain opens and saves in either editor without losing a field. Both editors use the same rule drawer and rules grid. The domain is chosen when the rule package is created and changed in its **Package meta** tab.

```
Metron: assessments (*.assessment.yaml) that name the Tetra model by meta.modelId
        |
        v  each assessment's requirement package lists rule packages in uses
Metron: the rule packages of those requirement packages
        |
        v  a run reads the pair and attribute rows of every such assessment
Record in the register of the rule package and model (the same register as a Tetra record)
```

- **Metron owns the rows, and the rule package owns the records.** A run reads the assessments of the model and writes only the register. It never writes an assessment or a Tetra file.
- **The model decides what a run covers.** A run covers one Tetra model, and it reads every assessment that names that model, whatever requirement package the assessment pins.
- **A link names the live versions.** A record stores the version of each Tetra entity and each requirement it depends on, and a hash of the outcome or of one attribute state. A change to one of them moves the record to Changed.
- **A run refreshes manual links.** A record that a person created under a manual rule gets its links resolved again, so a moved version shows as Changed.


See [Records in Metron](/docs/sightline/usage/metron/rule-findings).

## Workflow: record, cause (Tetra and Lamina)

This workflow links the architecture to the risk register without a Metron assessment.

```
Tetra: a rule package that the model names under uses
       |
       v  a run of the rules writes the records to the register
Tetra record (id: package, rule, model, then the rule's key)
       |
       v  a Lamina cause links it
Lamina: cause.vector holds the link and the hashes of what the cause accepted
       |
       v  Lamina compares the hashes with the record on load
Lamina: Current, Data changed, Source changed or Closed (a Problems-panel warning and a props drawer block)
       |
       v  the block accepts the change or removes the link
Lamina: the cause's vector, staged until the user saves
       |
       v  Lamina lists the records in the declared scope that no cause links
Lamina: the Tetra records list in the Findings dialog
       |
       v  the user creates a cause from a listed record
Lamina: the open editor stages a new cause with the link and the accepted hashes
```

- **Tetra owns the identity.** A record belongs to a rule package and a Tetra model. Tetra writes it to the register, and Metron takes no part.
- **The link is stored once, on the cause's vector.** It is a link with target kind `finding` and relationship `vector`. The accepted hashes sit beside it.
- **Lamina checks the hashes when the model opens.** It reads the register of each linked record through the storage server. A record that it cannot read shows the name the link holds.
- **The model declares the scope.** The Lamina manifest lists the Tetra zone ids it covers under `tetraZones`, and the one Tetra model it lists under `tetraModel`. The Findings dialog lists only the records of that model that enter those zones, and a cause covers one by linking its id. The Model Properties drawer in Lamina edits both fields.
- **Each app opens the other at the object.** Lamina offers **Open in Tetra** on a record block, and Tetra offers **Open** on a covering cause. An open window is reused, and a window that is still loading opens on the entity when the model is ready.
- **Tetra computes the back-reference.** Tetra scans every `*.bowtie.yaml` manifest in the workspace and reads the causes of the files each one lists. It lists the causes whose link names a record of one of its channels or flows, and stores nothing. The Tetra diagram shows the count as a cause badge, which the View menu can hide.
- **Accepting writes the cause only.** Accept stores the current hashes on the cause's vector. No register changes, and an Accept of the record in Metron leaves the cause's accepted state alone.

A cause can hold a Tetra Vector link with a snapshot. Lamina keeps both in the file and does not read them. Remove the link and link a record instead.

See [Record links](/docs/sightline/usage/lamina/finding-links) and [Jump between a cause and its Tetra channel or flow](/docs/sightline/usage/shared/finding-jump).

## Consequence categories (Lamina and reports)

Each risk matrix definition must declare its consequence categories. The declaration is a list of a `key`, a `label` and an optional `description` for each category, under `consequenceCategories`. The categories are the dimensions that Lamina rates for each outcome, for example safety and environment. This single list drives:

- The outcome box on the Lamina canvas, and the aggregated consequence grid of the event box.
- The consequence editor in the props drawer, which shows one row for each declared category.
- The worst-consequence and per-category rollups.
- The report generator. The PPTX report has one column for each declared category. The DOCX template maps the standard six categories by key, and it fills the remaining slots with custom keys in declared order.

Saved consequence data stays an open map keyed by category. Lamina drops a consequence on load when its key is not declared in the definition. The diagnostic names the outcome id and the key.

The built-in default definition declares the six standard categories. A custom definition must include a `consequenceCategories` block. Without the block, the definition fails validation and the risk data of that model is not usable. For example:

```yaml
consequenceCategories:
  - { key: safety, label: Safety }
  - { key: environment, label: Environment }
  - { key: community, label: Community }
  - { key: financial, label: Financial }
  - { key: workforce, label: Workforce }
  - { key: compliance, label: Compliance }
```
