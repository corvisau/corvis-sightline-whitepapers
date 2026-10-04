This page describes how Lamina, Tetra and Metron work together. It covers how the apps find each other's models, what each app may write, and the workflows that cross an app boundary.

## The apps

| App | Models | Files |
|---|---|---|
| Tetra | Architecture models: zones, devices, networks, channels, flows, information and data slices | `*.tetra.yaml` |
| Lamina | Risk models: causes, exposures, controls, consequences and the risk matrix | `*.bowtie.yaml` |
| Metron | Requirement packages, assessments, rules and findings | `*.metron.yaml`, `*.req.yaml`, `*.assessment.yaml`, `*.rules.yaml`, `*.finding.yaml` |
| Sightline | Coordinator: one activity-bar entry with a combined model tree over Tetra, Lamina and Metron | — |

Lamina, Tetra and Metron run inside the Sightline extension. The apps share the model files in the workspace.

### Shared authoring language

Lamina and Tetra resolve `template:` inheritance and Handlebars partials with the same engine. A modeller who moves between the apps meets one authoring language. Both apps also ship the same **Template Editor** and the same **Constants Editor**. See [Tetra templates](/docs/sightline/usage/tetra/templates).

## How the apps find each other

Two rules hold for every cross-app interaction.

1. **An app finds another app's files by scanning the workspace for well-known file patterns.** No setting lists another app's files. A new file is found the next time the app loads.
2. **Each app writes only its own model files, with one exception.** An app that owns a link writes that link's reference field into the other app's record, and it rewrites only that one file. Lamina writes `causeRef` onto a finding. Metron never writes a Lamina cause or control.

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

## Workflow: compliance assessment (Metron and Tetra)

Metron loads a requirement package and binds it to a Tetra model through an assessment file. The assessment file names the Tetra model in `meta.model`. Metron then resolves each pair of one requirement and one entity.

Each pair carries its outcome, evidence and notes in the assessment file. A pair has one of three built-in states:

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

Tetra reads the Metron findings and coverage of the model that it has open from the assessments in the workspace. These reads need the `view:metron` licence grant, and a refused read reports its reason. See [Licensing](/docs/sightline/usage/shared/licensing).

The storage server exposes the same findings and coverage, and the causes that cover a Tetra entity, as routes. See the [API reference](/docs/sightline/reference/api/overview).

### How Lamina reads Tetra Vectors

Lamina reads the Tetra Vectors of the Tetra models under the Lamina model's folder. It uses them for a cause's vector picker, for the Current, Changed and Gone check on a linked vector, and for the vectors in the declared zones that no cause links. None of these reads needs a Metron licence grant.

The storage server exposes the same three reads as routes. See the [Discovery routes](/docs/sightline/reference/api/discovery).

## Workflow: rule, finding, cause (Metron, Tetra and Lamina)

This workflow links a compliance gap to the risk register.

```
Metron: assessed pairs of requirement and entity
        |
        v  promote
Metron: finding in a findings register (a *.finding.yaml file in any folder)
        status: open, reviewed or resolved (you set it)
        refs: version-pinned references to the assessed pairs
        |
        v  Lamina: the findings inbox lists findings that have no cause
Lamina: Create cause from finding
        |  1. creates a draft cause
        |  2. writes causeRef onto the finding in its own file
        v
Lamina: the cause shows its linked findings, read only
```

A Metron user promotes assessed pairs into a **finding**. A Lamina user then promotes the finding into a **cause**.

- **A finding is a record that you edit.** Its status is `open`, `reviewed` or `resolved`, and you set it. Metron also computes a staleness check that compares the version that each reference pinned with the live pair.
- **The finding stores the one persisted link.** `causeRef` lives on the finding. A cause has no stored finding reference. Lamina scans the findings registers and shows the findings that point to the cause.
- **Metron ignores `causeRef`.** Metron keeps the field when it saves a finding. It does not mark a promoted finding differently from an open one.
- **Rules do not create findings.** The **Rules** tab is for authoring. The only live rule execution is the scope rule or the assessment rule that a requirement names. A rule narrows the scope or suggests an outcome.

### Findings on the Tetra diagram

Tetra draws a findings badge on zones, groups, devices, networks and information nodes. Hover over the badge to see the findings for that entity. The hover card offers three actions:

- **Open in Table** opens the Tabular Editor, filtered to the entity.
- **Open in Drawer** opens the props drawer of the entity, with a read-only Findings section.
- **Jump to Metron** opens the assessment that is bound to the finding, on the **Findings** tab, with that finding selected. When a package has more than one version, Metron uses the newest.

A second jump for the same assessment reuses the open tab and moves the selection to the new finding. Jump to Metron finds the assessment from the model id and package id that the finding stores. See [Findings overlay](/docs/sightline/usage/tetra/findings-overlay).

## Workflow: Tetra Vector, cause (Tetra and Lamina)

This workflow links the architecture to the risk register without a Metron assessment.

```
Tetra: a channel or flow that enters a zone from another zone
       narrowed by the vector rules the model declares
       |
       v  derived on every load, never stored
Tetra Vector (key: channel or flow id, then the zone it enters)
       |
       v  a Lamina cause links it
Lamina: cause.vector holds the link and a snapshot of the Tetra Vector
       |
       v  Lamina compares the snapshot with the live Tetra Vector on load
Lamina: Current (no entry), Changed or Gone (a Problems-panel warning and a props drawer banner)
       |
       v  the banner updates the snapshot, removes the link or keeps the vector as hand-written
Lamina: the cause's vector, staged until the user saves
       |
       v  Lamina lists the Tetra Vectors in the declared zones that no cause links
Lamina: the Tetra Vectors section of the Sightline sidebar
       |
       v  the user covers a listed vector from its context menu
Lamina: the open editor stages the link and snapshot on a new or existing cause
```

- **Tetra owns the identity.** A Tetra Vector is a view of the model. Tetra stores no list of them, and Metron takes no part.
- **The link is stored once, on the cause's vector.** It is an entity link with relationship `vector`. The snapshot sits beside it.
- **Lamina checks the snapshot when the model opens.** It reads the Tetra models in the manifest's folder and below. A model with no Tetra manifest is not checked.
- **The model declares the scope.** The Lamina manifest lists the Tetra zone ids it covers under `tetraZones`, and the one Tetra model it scans under `tetraModel`. The sidebar section lists only the Tetra Vectors of that model that enter those zones, and a cause covers one by linking its key. The Model Properties drawer in Lamina edits both fields.
- **Each app opens the other at the object.** Lamina offers **Open in Tetra** on a cause, and Tetra offers **Open** on a covering cause. An open window is reused, and a window that is still loading opens on the entity when the model is ready.
- **Tetra computes the back-reference.** Tetra scans every `*.bowtie.yaml` manifest in the workspace and reads the causes of the files each one lists. It lists the causes whose link names one of its Tetra Vectors, and stores nothing. The Tetra diagram shows the count as a cause badge, which the View menu can hide.
- **The section acts on what it lists.** A row's context menu creates a cause from the vector or links the vector to an existing cause. The open editor stages the link and the snapshot, so Save commits the change.
- **Rules carry the likelihood and the suggested controls.** A vector rule names a likelihood label and control template ids. Tetra never reads the risk matrix, so Lamina converts the label with the model's matrix and warns about a label or control id it cannot resolve. The snapshot holds both values.
- **Lamina writes the fix into the Tetra rule file.** The Quick Fix on an unresolved likelihood or control id opens a picker in the Lamina editor. Lamina then replaces the value in the `*.vector-rules.yaml` file of each rule that emitted the vector. The edit changes the rule, so every vector that rule emits changes with it.
- **Lamina resolves a Changed or Gone vector in the props drawer.** The banner stages one edit to the cause's vector: update the snapshot, remove the link, or keep the vector as hand-written. Tetra takes no part.

See [Tetra Vector links](/docs/sightline/usage/lamina/tetra-vector-links) and [Jump between a cause and its Tetra channel or flow](/docs/sightline/usage/shared/tetra-vector-jump).

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
