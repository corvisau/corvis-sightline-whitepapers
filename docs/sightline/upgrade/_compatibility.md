This page lists which versions of a model, a template and a command work together. It records the changes that can stop an older file from loading or from rendering.

A migration tool converts a model in the earlier format. Ask Corvis for the tool that matches your release.

This page is migration reference for the person who upgrades a model directory. The app does not show it, and the commands run from a shell.

## Tetra Schema Versions

A Tetra manifest carries a `schemaVersion`.

| Version | Shape | Status |
|---|---|---|
| 0 or 1, or absent with an older shape in the manifest | An older shape that Tetra no longer converts. | Does not open |
| Absent, with no older shape in the manifest | Read as version 2. | Opens |
| 2 | The shape of a model with no data slice that selects its own entities. | Opens |
| 3 | Version 2, and a data slice can carry `include` and `exclude`. | Opens, unless a slice uses `baseSlice` |
| 4 | Version 3, and no data slice uses `baseSlice`. A slice lists the slices it includes in `slices`. | Opens, unless a device lists `connections` |
| 5 | Version 4, and no device lists `connections`. A device is on a network through a channel. | Opens, unless a path `via` holds a glob |
| 6 | Version 5, and no path `via` holds a glob. A `via` names one channel that is drawn between the hop's two nodes. | Opens |
| 7 | The same shape as version 6. No model needs converting. | Opens |
| 8 | Version 7, and a path run with no `via` is `auto`: it follows a channel drawn between its two nodes, else draws direct. `via: direct` pins a direct line, so no channel may use the id `direct`. No model needs converting, and a model at version 7 opens and draws its runs as `auto`. | Opens |
| 9 | Version 8, and a `properties` value can be a list of text items. No model needs converting. | Opens |
| 10 | Version 9, and the manifest can carry a `canvasNotes` list of notes, each on one data slice. No model needs converting. | Opens |
| 11 | Version 10, and a note can carry a `colour`, one of eight presets. No model needs converting. A build from before version 11 refuses a note that has a `colour`. | Opens |
| 12 | Version 11, and a flow says which way the information moves with `infoDirection`, in place of `direction`. A flow `initiation` is `interactive` or `programmatic`. A flow can state `initiator` (`a`, `b` or `both`), the end that opens the exchange; no model needs converting for it. A channel keeps `direction`. A build from before version 12 reads a flow's `infoDirection` as the default, `bidirectional`. This is the shape that **Tetra: New Model** writes. | Current |

A model with no `schemaVersion` opens when its manifest holds none of the older shapes. Tetra does not add the key when it saves such a model.

A model that uses `baseSlice` does not open, whatever its `schemaVersion` says, and a model with no `schemaVersion` is no exception. The message names the data slice command below, which converts the model. A model that the stamp command marks at version 10 does not open in an older build of Sightline.

A model whose manifest holds an older shape does not open. The message names each older shape that the manifest holds and the conversion that applies to it. A manifest that says version 1 does not open either, because version 2 is the lowest that Tetra opens.

A model file can also hold an older shape that the manifest does not show, and a manifest can hold entities. Tetra does not open a model while the manifest or any other file holds one. The message names each file and shape and gives the command that converts it. Run these commands on the model folder, then stamp the model:

| Older shape | Command |
|---|---|
| A manifest that holds `tree`, `zones`, `systems`, `groups`, `devices`, `networks`, `channels`, `information`, `flows`, `annotations` or `overlays` | `pnpm --filter @sightline/tools-tetra-manifest-entities-migration run migrate --model-dir <model folder>` |
| A device with a `channels` or `networks` field | `pnpm --filter @sightline/tools-tetra-connections-migration run migrate --model-dir <model folder>`, then the command for a device with a `connections` field |
| A device with a `connections` field | `pnpm --filter @sightline/tools-tetra-synthetic-channels-migration run migrate --model-dir <model folder>` |
| A channel or flow path whose `via` holds a glob, such as `*` | `pnpm --filter @sightline/tools-tetra-path-via-migration run migrate --model-dir <model folder>` |
| An annotation with a `position`, `distance`, `maxWidth`, `positionOffset` or `severity` field, a link with a `position` or `positionOffset` field, a channel or flow with a `portOffset` field or a `showLabel` of `true` or `false`, or a zone, group, device, network or data slice with a `layout` of `horizontal` or `vertical` | `pnpm --filter @sightline/tools-tetra-field-migration run migrate --model-dir <model folder>` |
| A data slice with a `baseSlice` field | `pnpm --filter @sightline/tools-tetra-slice-migration run migrate --model-dir <model folder>` |
| A channel, or a device connection, with a `type` field | `pnpm --filter @sightline/tools-tetra-channel-type-migration run migrate --model-dir <model folder>` |
| A flow with a `direction` field, or a flow with an `initiation` of `offline` | `pnpm --filter @sightline/tools-tetra-flow-direction-migration run migrate --model-dir <model folder>` |
| A data slice program or a member `expr` that selects the kind of entity with `item.type`, such as `item.type == 'zone'`, in place of `item.kind` | `pnpm --filter @sightline/tools-item-kind-migration run migrate --model-dir <model folder>` |

Tetra no longer has a Zone Trust field. A model that still holds a `zoneTrust` or `zoneId` value on a zone opens without an error, and the value disappears the next time Tetra saves that file. No command converts it. To keep a value, copy it into the zone's description or a tag before you save.

The field command moves an annotation bubble that has a corner position, such as `bottom-left`, to the middle of that side. It changes a `layout` of `horizontal` to `columns` and `vertical` to `rows`.

The data slice command replaces `baseSlice` with `slices`, so a slice that built on another now inherits its layout. The slice runs its own filter inside the inherited slice, as before, because the command rewrites each filter line that starts from `model` to start from `inherited`. A slice with an `include` key now selects its included rows from the whole model, so it can show more. The command prints each such slice, so you can check it. It drops a `baseSlice` that named a slice that does not exist, the slice itself, or a loop of slices, because Tetra already ignored those, and prints each one.

The path via command makes every `via` in a channel or flow path name one channel that is drawn between the hop's two nodes: the channel's `nodeA` and its first `nodeB`, in either order. It rewrites a glob to the one channel that matches it and joins the two nodes, so `*` becomes that channel's id. It drops a glob that matches no channel or several, and a `via` that names a channel that does not join the two nodes, because the line did not follow a channel of that kind in the hop stacks. A dropped `via` draws that run directly, and the command prints each one, so you can pick a channel for the hop in the path field. A `via` that names a later end of a channel with several `nodeB` ends is dropped too.

The channel type command replaces `type` with a `channelType`, a `protocol` list and, for a wireless link, a `lineStyle` of `dashed`, so the line looks the same. It leaves a channel with no `type` as it is. It lists each channel of type `physical`, which has no equivalent, and each data slice line that compares `item.type` with a retired channel type. Remove or change those by hand.

After the conversions, run `pnpm --filter @sightline/tools-tetra-schema-stamp run migrate --model-dir <model folder>`. The command checks the whole model, writes `schemaVersion: 10` to the manifest, and lists the conversions that remain when it finds an older shape.

Tetra accepts any id made of letters, digits, dots, underscores and hyphens for a new group, device, network, information or system. An id such as `dev-erp-prod-app` needs no change.

A model whose `schemaVersion` is higher than this build supports does not open. The message says that the model is newer than the installed build and asks you to update Sightline. A `schemaVersion` that is not a whole number of 0 or higher also stops the model from opening, and the message names the value.

Each older shape has a conversion. Apply the conversions that suit a model directory once, and apply them before anything else loads the model.

| Older shape | Conversion |
|---|---|
| A flat `blockdiagram` with an `id` and a `label` and no `meta` | Moves the `id`, `label` and `description` under `meta`. |
| A model-level `layout` | Removes it, because a data slice sets the orientation. |
| An inline `members` list | Moves the inline members of a manifest into a tree file of their own. |
| A `views:` list | Folds it into the data slices. |
| A data slice filter written as a string, or collapsed containers listed in `collapsed` or `groups` | Converts string filters to program filters and the `collapsed` and `groups` options to `containers`. |

A conversion stamps `schemaVersion: 1` on a manifest when none of the older shapes is left in it. A second run changes nothing. A dry run lists the files without writing them. A conversion rewrites the file with a YAML serializer, so comments in a manifest it rewrites are not kept.

## Tetra Manifest Entities

Tetra does not open a model whose manifest holds an entity collection. The message names the manifest and the command below. A Tetra manifest holds model settings only: `schemaVersion`, `blockdiagram`, `files`, `imports`, `templates`, `uses`, `dataSlices`, `queries`, `reports` and `zoneColourDefs`. Every entity sits in a file that the manifest lists under `files`. A manifest that still holds `tree`, `zones`, `systems`, `groups`, `devices`, `networks`, `channels`, `information`, `flows`, `annotations` or `overlays` moves them with one command:

`pnpm --filter @sightline/tools-tetra-manifest-entities-migration run migrate --model-dir <model folder>`

The command moves each collection into the listed file that already holds that kind of entity. When no listed file holds it, the command creates a file named for the model and the kind, such as `plant.devices.yaml`, and adds it to `files`. The manifest `tree` goes to the model's tree file. The command never overwrites a file, and it adds `-2`, `-3` and so on to a name that is taken. A second run changes nothing. A dry run lists the files without writing them. The command rewrites the files it changes with a YAML serializer, so their comments are not kept.

When the same id is defined in the manifest and in a listed file, the command names the id and the files, leaves that model unchanged, and exits with an error. Remove one definition, then run the command again.

## Lamina Manifest ID

A Lamina manifest needs a `meta.id`. A manifest without one does not open, because the Models tree does not list it. The message that Lamina shows names the file and gives the command that adds an id to every manifest in its folder. See [Open a Lamina model](/docs/sightline/usage/lamina/opening-a-model).

## Lamina Schema Versions

A Lamina manifest carries a `schemaVersion`. Lamina reads a manifest with no `schemaVersion` as version 0.

| Version | Shape | Status |
|---|---|---|
| 0, or absent | Views, each of which can hold its own `aggregation`. | Opens |
| 1 | Views, with the aggregation profiles in a top-level `aggregations` list in place of the view. | Opens |
| 2 | Data slices in place of views, with the aggregation profiles still in `aggregations`. | Opens |
| 3 | Each Risk sets its own aggregation. See [Lamina schema version 3](#lamina-schema-version-3). | Opens |
| 4 | Risks that include elements, exclude elements and run a program, with no data slices. See [Lamina schema version 4](#lamina-schema-version-4). | Opens |
| 5 | A Risk carries `label` in place of `name`. See [Lamina schema version 5](#lamina-schema-version-5). | Opens |
| 6 | A cause has no hand-typed zones. See [Lamina schema version 6](#lamina-schema-version-6). | Opens |
| 7 | A cause holds its record links in `links` and its approved `snapshot`, has no actor, vector or exposure, and the `exposure-vector-grouped` mode is gone. See [Lamina schema version 7](#lamina-schema-version-7). | Current |

Lamina opens a manifest of any version from 0 to 7. It reads an older manifest as version 7, and it does not change the file when it opens it. The web shell and the API refuse a change to a model that still holds a key that version 3, version 4 or version 5 removed or renamed. Convert such a model with the commands in the sections below. A cause that still holds a field that version 6 removed or that version 7 moved is skipped when the model opens; see [Lamina schema version 6](#lamina-schema-version-6) and [Lamina schema version 7](#lamina-schema-version-7). Run the commands in order, from the lowest version that the model has.

A manifest whose `schemaVersion` is higher than 6 does not open. The message says that the model is newer than the installed build and asks you to update Sightline.

## Lamina Schema Version 3

Lamina schema version 3 moves aggregation to the Risk. A version 3 manifest has none of these keys:

- `aggregations`
- `riskAggregationId`
- `aggregationId` on a data slice
- `events`, `eventTags` and `dataSlice` on a Risk

A Risk lists `parts` in place of the last three. Each Risk can set an `aggregation`, and `riskDefaults` sets the aggregation of every Risk that sets none.

| Version 2 key | Version 3 form |
|---|---|
| The profile that `riskAggregationId` names | `riskDefaults`. The `likelihood` block becomes `perEvent.likelihood`, without `collapse`, and the `outcomes` mode becomes `perEvent.consequence`. |
| `aggregationId` of a data slice | Removed. With no `riskAggregationId` and no `riskDefaults`, the profile that the slices name becomes `riskDefaults`, and the ratings of the Risks can change. |
| Tag grouping in a profile | Removed. A grouped cause, event or outcome has no equivalent. |
| `events` of a Risk | One `event` part for each id. |
| `dataSlice` of a Risk | A `dataSlice` part. |
| `eventTags` of a Risk | A `dataSlice` part. The migration writes a data slice named `risk-<Risk id>-tags` with the filter `event.tags |> hasAll(...)`. |

A version 2 manifest opens. Lamina reads it as version 3, selects the same events for each Risk, and leaves the file as it is. The Risk routes of the API answer from the same reading. The `ratingBasis` of the Risk answers no longer has an `aggregationId` member, and the routes `GET`, `PUT` and `DELETE /models/{id}/aggregations` answer 404.

The web shell and the API refuse a change to the data slices or the manifest of a model that still holds a removed key. The message gives the command below. Run it once on the model folder:

```bash
pnpm --filter @sightline/tools-bowtie-risk-aggregation-migration run migrate --model-dir <model folder>
```

The command rewrites each `*.bowtie.yaml` file of the folder, writes `schemaVersion: 3` and keeps the comments and the order of the keys. It prints a warning for each profile feature that it cannot carry over, and for a model default that it took from the data slices. Add `--dry-run` to list the files without writing them. A second run changes nothing. A manifest below version 2, or with no `schemaVersion`, first takes the earlier migrations that Lamina applies when it opens such a model. A top-level key that those migrations change is written again in full, so a comment inside that key is lost. Views are no longer part of the manifest, so the command removes them and prints a warning that names each one. Re-create each view in Lamina.

A build of Sightline from before this release refuses a manifest of version 3.

## Lamina Schema Version 4

Lamina schema version 4 replaces data slices and parts-based Risks with one Risk form that includes elements, excludes elements and runs a program. A version 4 manifest has none of these:

- `dataSlices`
- `parts` on a Risk
- `dataSlice` on a report

| Version 3 form | Version 4 form |
|---|---|
| A data slice | A Risk named `slice-<id>`, in the `Data slices` risk folder, with the filter of the slice as its program and no aggregation. A filter that has no program form becomes a list of the elements the slice selects now. The view settings of the slice (`compact`, `showTags`, `showElements`, `outcomeSort` and `annotationDisplay`) have no equivalent and are dropped with a warning. |
| A Risk of `parts` | A Risk that includes its elements. An `event` part includes the event with the causes and outcomes that link to it. A `dataSlice` part holds the Risk of that slice, and a `risk` part stays. A Risk that holds a Risk keeps it as one rated item, so its ratings do not change. |
| `dataSlice` of a report | `risk`, which names the Risk of the slice. |

A model with no `risks` list used to have one Risk for each event. A version 4 model has none, and the Diagram shows a prompt to create a Risk. The read of the Risks no longer has the `mode` member, and a Risk no longer has `source`.

A version 3 manifest opens. Lamina reads it as version 4 and leaves the file as it is. The Risk routes of the API, the reports and the PNG export answer from the same reading. The `data-slices` routes of the API answer 404 for a Lamina model, and the routes of a Tetra model are unchanged.

The web shell and the API refuse a change to a model that still holds a removed key. The message gives the command below. Run it once on the model folder:

```bash
pnpm --filter @sightline/tools-bowtie-risk-tree-migration run migrate --model-dir <model folder>
```

The command rewrites each `*.bowtie.yaml` file of the folder, writes `schemaVersion: 4` and keeps the comments and the order of the keys. It prints a warning for each slice, Risk or report that it cannot carry over exactly. Add `--dry-run` to list the files without writing them. A second run changes nothing. The command skips a manifest below version 3 and says so. Run the version 3 command above first.

A build of Sightline from before this release refuses a manifest of version 4.

## Lamina Schema Version 5

Lamina schema version 5 names the display text of a Risk `label`, as every other entity does. A version 5 manifest has no `name` on a Risk. The `name` of the model (`meta.name`), of a report and of an aggregation is unchanged.

| Version 4 form | Version 5 form |
|---|---|
| `name` on a Risk | `label` |
| `name` in the body of a Risk query of the API | `label` |

A version 4 manifest opens. Lamina reads it as version 5 and leaves the file as it is, and the next save writes `label` and `schemaVersion: 5`. The Risk routes of the API, the reports and the PNG export answer from the same reading, and the answers carry `label`.

The web shell and the API refuse a change to a model whose Risks still say `name`. The message gives the command below. Run it once on the model folder:

```bash
pnpm --filter @sightline/tools-bowtie-risk-label-migration run migrate --model-dir <model folder>
```

The command rewrites each `*.bowtie.yaml` file of the folder, renames `name` to `label` on each Risk, writes `schemaVersion: 5` and keeps the comments and the order of the keys. A Risk that has both keys keeps its `label`, and the command prints a warning. Add `--dry-run` to list the files without writing them. A second run changes nothing. The command skips a manifest below version 4 and says so. Run the version 4 command above first.

A build of Sightline from before this release refuses a manifest of version 5.

## Lamina Schema Version 6

Lamina schema version 6 removes the zones that an author typed on a cause. A Tetra-linked cause takes its zones from its flows, and a cause that links no flow has none. A version 6 model has none of these:

- `zones` on a cause
- a zone link on a cause (`links` with `uriType: entity` and `linkType: zone`)
- `sourceZones` on a vector
- `zones` on an exposure

| Version 5 form | Version 6 form |
|---|---|
| A zone link on a cause | Removed. The cause draws no Source box until it links a Tetra flow. |
| `zones` on a cause | Removed. A preventative control no longer takes the zones of its cause, so a control states its own `zones`. |
| `sourceZones` on a vector | Removed. |
| `zones` on an exposure | Removed. |

A zone link on an event, a zone link on a control and the `zones` of a control, an outcome or an objective are unchanged.

Lamina skips a cause, vector or exposure that still holds a removed field when the model opens. The Problems panel names the field and the command below. Run the command once on the model folder:

```bash
pnpm --filter @sightline/tools-bowtie-zone-fields-migration run migrate --model-dir <model folder>
```

The command removes the four fields from every file of the model, writes `schemaVersion: 6` on a version 5 manifest and keeps the comments. It copies nothing. The `zones` of a cause is not moved to its controls, so a control with no `zones` of its own reports no assessment afterwards. The command prints each cause that had `zones` and each zone link it removed. It also prints each cause whose vector `sourceZones` and exposure `zones` differed. Add `--dry-run` to list the files without writing them. A second run changes nothing. The command leaves a manifest below version 5 as it is, so run the earlier commands first.

A build of Sightline from before this release refuses a manifest of version 6.

## Lamina Schema Version 7

Lamina schema version 7 simplifies the cause. The record links and the approved snapshot move off the vector onto the cause, and the cause loses its actor, vector and exposure.

| Version 6 form | Version 7 form |
|---|---|
| `links` on the vector of a cause | Added to the `links` of the cause, after the links it already holds. |
| `snapshot` on the vector of a cause | The `snapshot` of the cause. |
| The actor of a cause, inline or from an actor template | An item in the `actor` list property of the cause. |
| The vector and exposure of a cause that links a Tetra flow | Removed. The flows give the vector and the source zone. |
| The vector and exposure of a cause that links no flow | The text properties `vector` and `exposure` on the cause. |
| Another text or list key on the actor, vector or exposure, such as `intent` or `interfaces` | A property named `<object>-<key>` on the cause, such as `actor-intent`. |
| `suggestedControls` on the actor, vector or exposure | Added to the `suggestedControls` of the cause. |
| The `actors`, `vectors` and `exposures` lists, and their templates | Removed. |
| The aggregation mode `exposure-vector-grouped` | `exposure-grouped`. |

Lamina skips a cause that still holds an actor, vector or exposure when the model opens. It also drops a leftover `actors`, `vectors` or `exposures` list. The Problems panel names the command below, and the finding reads of the API answer with the same command. A manifest that still names `exposure-vector-grouped` does not open. Run the command once on the model folder:

```bash
pnpm --filter @sightline/tools-bowtie-cause-simplify-migration run migrate --model-dir <model folder>
```

The command converts every file of the model, writes `schemaVersion: 7` on a version 6 manifest and keeps the comments. A file left with nothing but the removed lists is deleted, and its entry leaves the manifest. A cause template converts the same way as a cause. Add `--dry-run` to list the files without writing them. A second run changes nothing. The command leaves a manifest below version 6 as it is, so run the earlier commands first.

The command prints what it could not carry over:

- A record link that the cause already held, a snapshot it kept, and a zone link it dropped.
- A value on the actor, vector or exposure that a property cannot hold, such as a nested object.
- A property that the cause already held under the name a value would take.
- Each aggregation mode it rewrote, with the Risk it sat in. Check the rating of that Risk.
- Each cause with a hand-typed exposure that links no flow. Under `exposure-grouped` it now shares one group with every cause that links no flow, so the rating of its Risk can change.

In the props drawer, Accept takes two undo steps, because it changes the links and the snapshot separately. Cause groups, the derived nodes that grouped causes with the same actor and exposure, no longer exist.

A build of Sightline from before this release refuses a manifest of version 7.

## Rule Package Domain

A rule package names the app that runs its rules in `meta.domain`. A rule no longer carries a `domain` of its own. A rule package manifest with no `meta.domain` does not load. Tetra and Metron report the package, and a run skips its rules.

Migrate a model directory once with the migration for this change.

The migration moves the per-rule `domain` of each package to `meta.domain`. A package whose rules name no domain gets `domain: metron`, and its rules then run in the Metron domain. A package whose rules name two domains is left unchanged, and the report names its rules. Move the rules of one domain to a package of their own, then run the migration again. The migration keeps the comments in the files that it rewrites, and a second run changes nothing. A dry run lists the files without writing them.

## Package Name

A Metron package manifest names the package in `meta.name`, and the manifest has no `meta.title`. Metron shows the name in the Models tree, in the editor heading and in the package pickers. A package manifest with no `meta.name` does not load. The message names the field. This applies to requirement, rule, attribute-set and scheme packages, and to the rule packages that Tetra reads.

Migrate a model directory once with the migration for this change.

The migration removes `meta.title` from every `*.metron.yaml` manifest under the directory. A manifest with no name takes its title as its name, or its id when it has no title. A title that differs from the name is reported, so you can move wording into the name before you keep the result. The migration changes only those lines, and a second run changes nothing. A dry run lists the files without writing them.

A report template variable `packageTitle` keeps its name and now holds the package name.

## Assessment Pairs

An assessment file (`*.assessment.yaml`) holds no `pairs`. The assessment's register holds each pair's state, outcome, evidence and notes, and Metron reads them from there. An assessment file that still holds `pairs` shows no outcomes, and the Problems panel warns about the list. Saving and running that assessment are refused, with a message that asks for the migration. The refusal protects the pairs, because a save would delete them.

Migrate a workspace once with the migration for this change.

The migration writes each pair into the register of its assessment and then removes `pairs` from the file. Run it on the workspace root, the folder that holds `findings/`. An assessment whose register is damaged is skipped with a warning, and its file keeps its pairs. The migration rewrites an assessment file in the standard layout, so comments in that file are not kept. A second run changes nothing. A dry run lists the files without writing them.

## Moving Old Assessments to the Shared Register

An assessment file that still holds a `sharedAssessments` list is not read for it. The Problems panel warns about the list, and Metron refuses to save the file so the list is not lost. The app has no action for this move. Run the backfill command once on the workspace root, the folder that holds `findings/`:

```bash
pnpm --filter @sightline/tools-metron-pairs-records-backfill run backfill --model-dir <workspace root>
```

The command moves each shared assessment into the register of its assessment and removes the list from the file. Add `--dry-run` to list the files it would write without changing them. An assessment whose register is damaged is skipped with a warning, and its file keeps its list. A second run changes nothing.

## Report Templates

The template field `perFilterSummary` is removed. A template must use `perSliceSummary`. A user-authored Word or Excel template that still uses `perFilterSummary` renders an empty result until you update it.

## Report Command Flags

| Flag | Status | Replacement |
|---|---|---|
| `--filter=<expr>` | Removed. The command ignores it without a warning. | `--data-slice=<id>` |
| `--aggregation=<id>` | Removed. The command ignores it without a warning. | `--data-slice=<id>`, because a data slice bundles the aggregation |
| `--filter-label=<text>` | Kept. It overrides the "Filtered by:" caption for display only. | Not applicable |
| `--data-slice=<id>` | New in version 2. It overrides the data slice that the report declares. | Not applicable |
| `--view-json=<json>` | New in version 2. It gives the renderer a display-only view. | Not applicable |

A script that passes a removed flag does not fail. The command drops the flag, so the report can differ from what the script intended. Change the script to use `--data-slice`.

## Metron Packages

A Metron assessment pins its package by the package id and the package version. A new version of a package starts with no assessments. See [Component packages](/docs/sightline/usage/metron/component-packages).

A `uses:` entry in a requirement package that names a package without a `version` becomes ambiguous when the package has two versions. Pin the entry to the version that you want.

## Model Folders

A new model of any app is written into a folder of its own, named after the model with the ending `.sightline`. A model made before that change keeps its files directly in its folder. Both kinds open and work. Only a model in its own folder can share a folder with other models.

The command below moves each such model into its own folder:

```
pnpm --filter @sightline/tools-model-folder-migration run migrate --model-dir <model folder>
```

The command creates the folder inside the folder that holds the model, and moves every file of the model into it. The model stays in the same folder in the **Models** tree. The command changes no file content, and no id changes, so every reference to the model stays valid. Add `--dry-run` to list the models without moving any file. A second run changes nothing.

The command leaves a model where it is, and prints a warning that names it, in these cases:

- The folder holds another model. Move one model into a folder of its own, then run the command again.
- The model is in the workspace root. Move it into a folder first.
- A file of the model names a path that leaves its folder, such as `../shared/types.yaml`. Move that file into the model's folder, or leave the model as it is.
- A folder for the model, named after it with the ending `.sightline`, already exists. A run that stopped part way can leave this state. Move the remaining files into that folder by hand, with the model's manifest file last.

## Licence Grants

A licence names the apps that it unlocks with the grants `edit:lamina`, `edit:tetra` and `edit:metron`. A bare `edit` grant is retired and unlocks nothing. The `view:metron` grant covers the Tetra findings badges and coverage overlay. A licence issued without it needs reissuing for those two features to load. See [Licensing](/docs/sightline/usage/shared/licensing).

## Related

- [Rollback](/docs/sightline/upgrade/rollback)
