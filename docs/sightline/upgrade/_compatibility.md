This page lists which versions of a model, a template and a command work together. It records the changes that can stop an older file from loading or from rendering.

A migration tool converts a model in the earlier format. Ask Corvis for the tool that matches your release.

This page is migration reference for the person who upgrades a model directory. The app does not show it, and the commands run from a shell.

## Tetra schema versions

A Tetra manifest carries a `schemaVersion`.

| Version | Shape | Status |
|---|---|---|
| 0 or 1, or absent with an older shape in the manifest | An older shape that Tetra no longer converts. | Does not open |
| Absent, with no older shape in the manifest | Read as version 2. | Opens |
| 2 | The shape of a model with no data slice that selects its own entities. | Opens |
| 3 | Version 2, and a data slice can carry `include` and `exclude`. | Opens, unless a slice uses `baseSlice` |
| 4 | Version 3, and no data slice uses `baseSlice`. A slice lists the slices it includes in `slices`. This is the shape that **Tetra: New Model** writes. | Current |

A model with no `schemaVersion` opens when its manifest holds none of the older shapes. Tetra does not add the key when it saves such a model.

A model that uses `baseSlice` does not open, whatever its `schemaVersion` says, and a model with no `schemaVersion` is no exception. The message names the data slice command below, which converts the model to version 4. A model that the command stamps at version 4 does not open in an older build of Sightline.

A model whose manifest holds an older shape does not open. The message names each older shape that the manifest holds and the conversion that applies to it. A manifest that says version 1 does not open either, because version 2 is the lowest that Tetra opens.

A model file can also hold an older shape that the manifest does not show, and a manifest can hold entities. Tetra does not open a model while the manifest or any other file holds one. The message names each file and shape and gives the command that converts it. Run these commands on the model folder, then stamp the model:

| Older shape | Command |
|---|---|
| A manifest that holds `tree`, `zones`, `systems`, `groups`, `devices`, `networks`, `channels`, `information`, `flows`, `annotations` or `overlays` | `pnpm --filter @sightline/tools-tetra-manifest-entities-migration run migrate --model-dir <model folder>` |
| A zone with a `zoneId` field | `pnpm --filter @sightline/tools-tetra-zone-trust-migration run migrate --model-dir <model folder>` |
| A device with a `channels` or `networks` field | `pnpm --filter @sightline/tools-tetra-connections-migration run migrate --model-dir <model folder>` |
| An annotation with a `position`, `distance`, `maxWidth`, `positionOffset` or `severity` field, a link with a `position` or `positionOffset` field, a channel or flow with a `portOffset` field or a `showLabel` of `true` or `false`, or a zone, group, device, network or data slice with a `layout` of `horizontal` or `vertical` | `pnpm --filter @sightline/tools-tetra-field-migration run migrate --model-dir <model folder>` |
| A data slice with a `baseSlice` field | `pnpm --filter @sightline/tools-tetra-slice-migration run migrate --model-dir <model folder>` |
| A channel, or a device connection, with a `type` field | `pnpm --filter @sightline/tools-tetra-channel-type-migration run migrate --model-dir <model folder>` |
| A data slice program or a member `expr` that selects the kind of entity with `item.type`, such as `item.type == 'zone'`, in place of `item.kind` | `pnpm --filter @sightline/tools-item-kind-migration run migrate --model-dir <model folder>` |

The field command moves an annotation bubble that has a corner position, such as `bottom-left`, to the middle of that side. It changes a `layout` of `horizontal` to `columns` and `vertical` to `rows`.

The data slice command replaces `baseSlice` with `slices`, so a slice that built on another now lists it. A slice with no filter, `include` or `exclude` of its own shows the same model as before. A slice with one of these did run it inside the base slice, and now shows the base slice plus whatever its own selection matches over the whole model, so it can show more. The command prints each such slice, so you can check it. It drops a `baseSlice` that named a slice that does not exist, the slice itself, or a loop of slices, because Tetra already ignored those, and prints each one.

The channel type command replaces `type` with a `channelType`, a `protocol` list and, for a wireless link, a `lineStyle` of `dashed`, so the line looks the same. It leaves a channel with no `type` as it is. It lists each channel of type `physical`, which has no equivalent, and each data slice line that compares `item.type` with a retired channel type. Remove or change those by hand.

After the conversions, run `pnpm --filter @sightline/tools-tetra-schema-stamp run migrate --model-dir <model folder>`. The command checks the whole model, writes `schemaVersion: 4` to the manifest, and lists the conversions that remain when it finds an older shape.

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

## Tetra manifest entities

Tetra does not open a model whose manifest holds an entity collection. The message names the manifest and the command below. A Tetra manifest holds model settings only: `schemaVersion`, `blockdiagram`, `files`, `imports`, `templates`, `uses`, `dataSlices`, `queries`, `reports` and `zoneColourDefs`. Every entity sits in a file that the manifest lists under `files`. A manifest that still holds `tree`, `zones`, `systems`, `groups`, `devices`, `networks`, `channels`, `information`, `flows`, `annotations` or `overlays` moves them with one command:

`pnpm --filter @sightline/tools-tetra-manifest-entities-migration run migrate --model-dir <model folder>`

The command moves each collection into the listed file that already holds that kind of entity. When no listed file holds it, the command creates a file named for the model and the kind, such as `plant.devices.yaml`, and adds it to `files`. The manifest `tree` goes to the model's tree file. The command never overwrites a file, and it adds `-2`, `-3` and so on to a name that is taken. A second run changes nothing. A dry run lists the files without writing them. The command rewrites the files it changes with a YAML serializer, so their comments are not kept.

When the same id is defined in the manifest and in a listed file, the command names the id and the files, leaves that model unchanged, and exits with an error. Remove one definition, then run the command again.

## Lamina manifest id

A Lamina manifest needs a `meta.id`. A manifest without one does not open, because the Models tree does not list it. The message that Lamina shows names the file and gives the command that adds an id to every manifest in its folder. See [Open a Lamina model](/docs/sightline/usage/lamina/opening-a-model).

## Lamina schema versions

A Lamina manifest carries a `schemaVersion`. Lamina reads a manifest with no `schemaVersion` as version 0.

| Version | Shape | Status |
|---|---|---|
| 0, or absent | Views, each of which can hold its own `aggregation`. | Opens |
| 1 | Views, with the aggregation profiles in a top-level `aggregations` list in place of the view. | Opens |
| 2 | Data slices in place of views, with the aggregation profiles still in `aggregations`. | Opens |
| 3 | Each Risk sets its own aggregation. See [Lamina schema version 3](#lamina-schema-version-3). | Opens |
| 4 | Risks that include elements, exclude elements and run a program, with no data slices. See [Lamina schema version 4](#lamina-schema-version-4). | Current |

Lamina opens a manifest of any version from 0 to 4. It reads an older manifest as version 4, and it does not change the file when it opens it. The web shell and the API refuse a change to a model that still holds a key that version 3 or version 4 removed. Convert such a model with the commands in the next two sections. Run the version 3 command first on a model below version 3, then the version 4 command.

A manifest whose `schemaVersion` is higher than 4 does not open. The message says that the model is newer than the installed build and asks you to update Sightline.

## Lamina schema version 3

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

## Lamina schema version 4

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

## Rule package domain

A rule package names the app that runs its rules in `meta.domain`. A rule no longer carries a `domain` of its own. A rule package manifest with no `meta.domain` does not load. Tetra and Metron report the package, and a run skips its rules.

Migrate a model directory once with the migration for this change.

The migration moves the per-rule `domain` of each package to `meta.domain`. A package whose rules name no domain gets `domain: metron`, and its rules then run in the Metron domain. A package whose rules name two domains is left unchanged, and the report names its rules. Move the rules of one domain to a package of their own, then run the migration again. The migration keeps the comments in the files that it rewrites, and a second run changes nothing. A dry run lists the files without writing them.

## Package name

A Metron package manifest names the package in `meta.name`, and the manifest has no `meta.title`. Metron shows the name in the Models tree, in the editor heading and in the package pickers. A package manifest with no `meta.name` does not load. The message names the field. This applies to requirement, rule, attribute-set and scheme packages, and to the rule packages that Tetra reads.

Migrate a model directory once with the migration for this change.

The migration removes `meta.title` from every `*.metron.yaml` manifest under the directory. A manifest with no name takes its title as its name, or its id when it has no title. A title that differs from the name is reported, so you can move wording into the name before you keep the result. The migration changes only those lines, and a second run changes nothing. A dry run lists the files without writing them.

A report template variable `packageTitle` keeps its name and now holds the package name.

## Assessment pairs

An assessment file (`*.assessment.yaml`) holds no `pairs`. The assessment's register holds each pair's state, outcome, evidence and notes, and Metron reads them from there. An assessment file that still holds `pairs` shows no outcomes, and the Problems panel warns about the list. Saving and running that assessment are refused, with a message that asks for the migration. The refusal protects the pairs, because a save would delete them.

Migrate a workspace once with the migration for this change.

The migration writes each pair into the register of its assessment and then removes `pairs` from the file. Run it on the workspace root, the folder that holds `findings/`. An assessment whose register is damaged is skipped with a warning, and its file keeps its pairs. The migration rewrites an assessment file in the standard layout, so comments in that file are not kept. A second run changes nothing. A dry run lists the files without writing them.

## Moving old assessments to the shared register

An assessment file that still holds a `sharedAssessments` list is not read for it. The Problems panel warns about the list, and Metron refuses to save the file so the list is not lost. The app has no action for this move. Run the backfill command once on the workspace root, the folder that holds `findings/`:

```bash
pnpm --filter @sightline/tools-metron-pairs-records-backfill run backfill --model-dir <workspace root>
```

The command moves each shared assessment into the register of its assessment and removes the list from the file. Add `--dry-run` to list the files it would write without changing them. An assessment whose register is damaged is skipped with a warning, and its file keeps its list. A second run changes nothing.

## Report templates

The template field `perFilterSummary` is removed. A template must use `perSliceSummary`. A user-authored Word or Excel template that still uses `perFilterSummary` renders an empty result until you update it.

## Report command flags

| Flag | Status | Replacement |
|---|---|---|
| `--filter=<expr>` | Removed. The command ignores it without a warning. | `--data-slice=<id>` |
| `--aggregation=<id>` | Removed. The command ignores it without a warning. | `--data-slice=<id>`, because a data slice bundles the aggregation |
| `--filter-label=<text>` | Kept. It overrides the "Filtered by:" caption for display only. | Not applicable |
| `--data-slice=<id>` | New in version 2. It overrides the data slice that the report declares. | Not applicable |
| `--view-json=<json>` | New in version 2. It gives the renderer a display-only view. | Not applicable |

A script that passes a removed flag does not fail. The command drops the flag, so the report can differ from what the script intended. Change the script to use `--data-slice`.

## Metron packages

A Metron assessment pins its package by the package id and the package version. A new version of a package starts with no assessments. See [Component packages](/docs/sightline/usage/metron/component-packages).

A `uses:` entry in a requirement package that names a package without a `version` becomes ambiguous when the package has two versions. Pin the entry to the version that you want.

## Licence grants

A licence names the apps that it unlocks with the grants `edit:lamina`, `edit:tetra` and `edit:metron`. A bare `edit` grant is retired and unlocks nothing. The `view:metron` grant covers the Tetra findings badges and coverage overlay. A licence issued without it needs reissuing for those two features to load. See [Licensing](/docs/sightline/usage/shared/licensing).

## Related

- [Rollback](/docs/sightline/upgrade/rollback)
