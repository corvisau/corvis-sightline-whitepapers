This page describes how Lamina, Tetra and Metron work together. It covers how the apps find each other's models, what each app may change, and the workflows that cross an app boundary.

## The apps

| App | Models |
|---|---|
| Tetra | Architecture models: zones, devices, networks, channels, flows, information and data slices |
| Lamina | Risk models: causes, exposures, controls, consequences and the risk matrix |
| Metron | Requirement packages, assessments, rules and findings |
| Sightline | Coordinator: one activity-bar entry with a combined model tree over Tetra, Lamina and Metron |

Lamina, Tetra and Metron run inside the Sightline extension. The apps share the models in the workspace.

### Shared authoring language

Lamina and Tetra resolve template inheritance and Handlebars partials with the same engine. A modeller who moves between the apps meets one authoring language. Both apps also ship the same **Template Editor** and the same **Constants Editor**. See [Tetra templates](/docs/sightline/usage/tetra/templates).

## How the apps find each other

Two rules hold for every cross-app interaction.

1. **An app finds another app's models by scanning the workspace.** No setting lists another app's models. A new model is found the next time the app loads.
2. **Each app changes only its own models.** An app that owns a link writes that link in its own record only. Lamina never changes a record or a finding, and Metron never changes a Lamina cause or control.

A cross-app link is a plain reference. The side that owns the lifecycle of the relationship stores it. The other side resolves it when it reads, as a computed back-reference that is never saved. This design has two effects:

- A reference is never stored on both sides, so the two sides cannot drift apart.
- A reference that no longer resolves shows as "—". It does not stop the model from loading.

### Model identity

Every model has a stable id that does not depend on where the model sits in the workspace. Ids are unique within each kind of model. A Lamina id and a Tetra id can share a string without a conflict. Metron compares the version of a package as well as its id, and it tells requirement packages and rule packages apart, so the two kinds can share an id. Every Metron outcome record and finding reference carries the model id, so two models that share an entity id never collide.

### Renaming, moving and deleting a model

A Tetra or Lamina model can be renamed, moved to another folder and deleted. Rename changes only the name that the model is listed under. Move re-files the model and changes no id, so every reference to the model keeps resolving. A Metron assessment finds its Tetra model by id, and a Lamina model finds its Tetra model the same way.

Delete is refused while another item still names the model's id. The refusal lists each referrer, and nothing is removed. These items count as referrers:

- A findings register for the model.
- A Metron assessment that binds the model.
- A Lamina model that scopes to the model.

Move and delete act on the whole directory of a model, so both are refused for a directory that holds another model, and for the workspace root. A move is also refused when the target directory exists, or when the model reads files outside its own directory.

A folder can be renamed, moved and deleted as well. Rename and move change no id, so no reference changes. Delete is refused while the folder holds any file, so a folder is removed only when it is empty. Rename and move are refused for the findings folder and everything under it, for a folder inside a model's directory, and for a folder that holds a Metron package. A move is also refused when a model in the folder reads files outside it, and when the destination exists.

## Workflow: compliance assessment (Metron and Tetra)

Metron loads a requirement package and binds it to a Tetra model through an assessment. The assessment names the Tetra model by its id. Metron finds the model from that id each time it loads the assessment, so the model can move to another folder without an edit to the assessment. Metron then resolves each pair of one requirement and one entity.

A reference from one model to another holds an id and never a folder or a path.

Each pair carries its outcome, state, evidence and notes in the assessment's register. The register holds one record for each requirement and in-scope entity, and the assessment holds no pairs of its own. The worklist, roll-ups, coverage, exports and reports read the register's records. A pair has one of three built-in states:

| State | Meaning |
|---|---|
| `draft` | The pair is editable. An automatic outcome from a rule starts as `draft`, so a machine suggestion never counts as a human sign-off. |
| `in-progress` | The pair is being worked and is editable. |
| `done` | The pair is read-only until you reopen it. Reopening adds an entry to the assessment log. |

Metron does not log outcome edits. Version control records those.

Suppose the requirement or the entity changes after a pair is `done`. Metron then flags the pair as needing revalidation. The flag shows as the **Reval** column of the worklist. The pair stays `done` and still counts toward coverage. A person clears the flag by reopening the pair or by choosing **Revalidate**.

The worklist narrows in two independent ways, and they combine with AND:

- A compliance slice, which is a named filter over categories, tags, target kinds and entity scope. It never changes what Metron assesses. The requirement's scope rule controls that.
- A filter on the worklist view. See [Worklist filtering](/docs/sightline/usage/metron/worklist-filtering).

### Coverage overlay in Tetra

Tetra can draw the assessment coverage of each entity on its diagram. The overlay is optional. Tetra computes the figures with Metron's rules from every assessment that names the Tetra model. The figures therefore equal the worklist and the Portfolio Roll-Up, and nothing is saved. When the licence does not name Metron data, the overlay shows a notice that gives the reason. See [Coverage overlay](/docs/sightline/usage/tetra/coverage-overlay).

### How Tetra reads Metron data

Tetra reads the Metron findings and coverage of the model that it has open from the assessments in the workspace. These reads need the Metron view grant of the licence, and a refused read reports its reason. A rule package is read with the grant of its domain: Tetra's own rule packages, and the findings their rules produce, need only the Tetra grant. The Metron findings that Tetra reads for its badges stay on the Metron view grant. See [Licensing](/docs/sightline/usage/shared/licensing).

The storage server exposes the same findings and coverage, and the causes that cover a Tetra entity. See the [API reference](/docs/sightline/reference/api/overview).

### How Lamina reads Tetra records

Lamina reads the records that the rule packages of a Tetra model create. It uses them for the state of each record that a cause links, and for the records in the declared scope that no cause links. It also lists the Tetra models near the Lamina model, which the Model Properties drawer offers as the choices for the Tetra model and the Tetra zones.

The host reads these answers from the storage server by the Lamina model's own id. A Lamina model outside the first workspace folder, or one that shares its id with another model, has no id to read by, so the host shows no record data for it. Lamina reads four answers:

- The Tetra models of the workspace, each with the zones it declares.
- The state of every record that a cause of the model links, such as current, changed or unreadable.
- The same states for one cause.
- The records in the model's scope, each marked linked or not.

The list of Tetra models needs no Metron licence grant. The three record reads follow the server's read check, as every finding read does. See the [Discovery routes](/docs/sightline/reference/api/discovery).

## Workflow: rule, finding, cause (Metron, Tetra and Lamina)

This workflow links a compliance gap to the risk register.

```
Metron: assessed pairs of requirement and entity
        |
        v  promote
Metron: finding in the register of the Tetra model
        status: open, reviewed or resolved (you set it)
        members: links to the assessed pairs, each pinned
        |
        v  Lamina: the findings inbox lists Metron findings that no cause links
Lamina: Create cause from finding
        |  creates a draft cause that holds the link
        |  pinned at the state of the finding that it accepted
        v
Lamina: the cause shows the state of its linked finding
```

A Metron user groups assessed pairs into a **finding**. A Lamina user then creates a **cause** that links the finding.

- **A finding is a record that you edit.** Its status is `open`, `reviewed` or `resolved`, and you set it. A run of the model's Metron rules refreshes its links, and the finding shows **Source changed** when a version moved.
- **A member is a link to an assessed pair.** The link names the pair and the assessment that holds it, so a finding can hold pairs of several assessments of the model.
- **The cause stores the one persisted link.** The accepted pin sits on the link. The link names the finding and the Tetra model that holds it. A finding stores nothing about a cause. Lamina computes where a finding is used from the links of its models.
- **Metron accepts a finding; Tetra only reads it.** **Accept** in Metron's **Findings** tab pins the links of the finding and records the acceptance in the finding's register. Tetra's **Records** tab lists the same record and changes none.
- **Lamina writes only its own model.** Creating the cause and accepting a change commit the cause, with the model's other staged changes, on **Save all**.
- **Scope and assessment rules do not create findings.** The **Rules** tab is for authoring. A scope rule or an assessment rule that a requirement names narrows the scope or suggests an outcome. A rule in a package of the Metron domain creates records instead. See [Records from assessed pairs](#workflow-records-from-assessed-pairs-metron).

### Findings on the Tetra diagram

Tetra draws a findings badge on zones, groups, devices, networks and information nodes. Hover over the badge to see the findings for that entity. The hover card offers three actions:

- **Open in Table** opens the Tabular Editor, filtered to the entity.
- **Open in Drawer** opens the props drawer of the entity, with a read-only Findings section.
- **Jump to Metron** opens the assessment that is bound to the finding, on the **Findings** tab, with that finding selected. When a package has more than one version, Metron uses the newest.

A second jump for the same assessment reuses the open tab and moves the selection to the new finding. See [Findings overlay](/docs/sightline/usage/tetra/findings-overlay).

## Workflow: records from assessed pairs (Metron)

A Metron rule in a rule package of the Metron domain creates records from the assessed pairs and attributes of one Tetra model. Tetra and Metron read a rule with one schema, so a package of either domain opens and saves in either editor without losing a field. Both editors use the same rule drawer and rules grid. A person chooses the domain when creating the rule package and changes it in the **Package meta** tab.

```
Metron: assessments that name the Tetra model
        |
        v  each assessment's requirement package lists rule packages
Metron: the rule packages of those requirement packages
        |
        v  a run reads the pair and attribute rows of every such assessment
Record in the register of the rule package and model (the same register as a Tetra record)
```

- **Metron owns the rows, and the rule package owns the records.** A run reads the assessments of the model and writes only the register. It never changes an assessment or a Tetra model.
- **The model decides what a run covers.** A run covers one Tetra model, and it reads every assessment that names that model, whatever requirement package the assessment pins.
- **A link names the live versions.** A record stores the version of each Tetra entity and each requirement it depends on, and a hash of the outcome or of one attribute state. A change to one of them moves the record to Changed.
- **A run refreshes manual links.** A record that a person created under a manual rule gets its links resolved again, so a moved version shows as Changed.

See [Records in Metron](/docs/sightline/usage/metron/rule-findings).

## Workflow: record, cause (Tetra and Lamina)

This workflow links the architecture to the risk register without a Metron assessment.

```
Tetra: a rule package that the model uses
       |
       v  a run of the rules writes the records to the register
Tetra record (identified by package, rule, model, then the rule's key)
       |
       v  a Lamina cause links it
Lamina: the cause holds the link and the hashes of what the cause accepted
       |
       v  Lamina compares the hashes with the record on load
Lamina: Current, Data changed, Source changed or Closed (a Problems-panel warning and a props drawer block)
       |
       v  the block accepts the change or removes the link
Lamina: the cause, staged until the user saves
       |
       v  Lamina lists the records in the declared scope that no cause links
Lamina: the Tetra records list in the Findings dialog
       |
       v  the user creates a cause from a listed record
Lamina: the open editor stages a new cause with the link and the accepted hashes
```

- **Tetra owns the identity.** A record belongs to a rule package and a Tetra model. Tetra writes it to the register, and Metron takes no part.
- **The link is stored once, on the cause.** The accepted hashes sit beside it.
- **Lamina checks the hashes when the model opens.** It reads the register of each linked record through the storage server. A record that it cannot read shows the name the link holds.
- **The model declares the scope.** A Lamina model lists the Tetra zones it covers and the one Tetra model it covers. The Findings dialog lists only the records of that model that enter those zones, and a cause covers one by linking it. The Model Properties drawer in Lamina edits both settings.
- **Each app opens the other at the object.** Lamina offers **Open in Tetra** on a record block, and Tetra offers **Open** on a covering cause. An open window is reused, and a window that is still loading opens on the entity when the model is ready.
- **Tetra computes the back-reference.** Tetra scans every Lamina model in the workspace and reads its causes. It lists the causes whose link names a record of one of its channels or flows, and stores nothing. The Tetra diagram shows the count as a cause badge, which the View menu can hide.
- **Accepting writes the cause only.** Accept stores the current hashes on the cause. No register changes, and an Accept of the record in Metron leaves the cause's accepted state alone.

A cause can hold an older Tetra link that Lamina keeps and does not read. Remove that link and link a record instead.

See [Record links](/docs/sightline/usage/lamina/finding-links) and [Jump between a cause and its Tetra channel or flow](/docs/sightline/usage/shared/finding-jump).

## Consequence categories (Lamina and reports)

Each risk matrix definition must declare its consequence categories. Each category has an identifier, a label and an optional description. The categories are the dimensions that Lamina rates for each outcome, for example safety and environment. This single list drives:

- The outcome box on the Lamina canvas, and the aggregated consequence grid of the event box.
- The consequence editor in the props drawer, which shows one row for each declared category.
- The worst-consequence and per-category rollups.
- The report generator. The PPTX report has one column for each declared category. The DOCX template maps the standard six categories by identifier, and it fills the remaining slots with custom categories in declared order.

Saved consequence data is keyed by category. Lamina drops a consequence on load when its category is not declared in the definition. The diagnostic names the outcome and the category.

The built-in default definition declares the six standard categories: safety, environment, community, financial, workforce and compliance. A custom definition must declare its categories. Without them, the definition fails validation and the risk data of that model is not usable.
