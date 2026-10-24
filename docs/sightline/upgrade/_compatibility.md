This page lists which versions of a model, a template and a command work together. It records the changes that can stop an older file from loading or from rendering.

A migration tool converts a model in the earlier format. Ask Corvis for the tool that matches your release.

This page is migration reference for the person who upgrades a model directory. The app does not show it, and the commands run from a shell.

## Lamina schema versions

A Lamina manifest carries a `schemaVersion`.

| Version | Shape | Status |
|---|---|---|
| 0 or 1, or absent with an older shape in the manifest | An older shape that Tetra no longer converts. | Does not open |
| Absent, with no older shape in the manifest | Read as version 2. | Opens |
| 2 | The shape that **Tetra: New Model** writes. | Current |

A model with no `schemaVersion` opens when its manifest holds none of the older shapes. Tetra does not add the key when it saves such a model.

A model whose manifest holds an older shape does not open. The message names each older shape that the manifest holds and the conversion that applies to it. A manifest that says version 1 does not open either, because version 2 is the lowest that Tetra opens.

A model file can also hold an older shape that the manifest does not show. Tetra does not open a model while any of its files holds one. The message names each file and shape and gives the command that converts it. Run these commands on the model folder, then stamp the model:

| Older shape | Command |
|---|---|
| A zone with a `zoneId` field | `pnpm --filter @sightline/tools-tetra-zone-trust-migration run migrate --model-dir <model folder>` |
| A device with a `channels` or `networks` field | `pnpm --filter @sightline/tools-tetra-connections-migration run migrate --model-dir <model folder>` |
| An annotation with a `position`, `distance`, `maxWidth`, `positionOffset` or `severity` field, or a link with a `position` or `positionOffset` field | `pnpm --filter @sightline/tools-tetra-field-migration run migrate --model-dir <model folder>` |
| A channel or flow with a `portOffset` field or a `showLabel` of `true` or `false` | `pnpm --filter @sightline/tools-tetra-field-migration run migrate --model-dir <model folder>` |
| A zone, group, device, network or data slice with a `layout` of `horizontal` or `vertical` | `pnpm --filter @sightline/tools-tetra-field-migration run migrate --model-dir <model folder>` |
| A data slice program or a member `expr` that selects the kind of entity with `item.type`, such as `item.type == 'zone'`, in place of `item.kind` | `pnpm --filter @sightline/tools-item-kind-migration run migrate --model-dir <model folder>` |

The field command moves an annotation bubble that has a corner position, such as `bottom-left`, to the middle of that side. It changes a `layout` of `horizontal` to `columns` and `vertical` to `rows`.

After the conversions, run `pnpm --filter @sightline/tools-tetra-schema-stamp run migrate --model-dir <model folder>`. The command checks the whole model, writes `schemaVersion: 2` to the manifest, and lists the conversions that remain when it finds an older shape.

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

## Lamina manifest id

A Lamina manifest needs a `meta.id`. A manifest without one does not open, because the Models tree does not list it. The message that Lamina shows names the file and gives the command that adds an id to every manifest in its folder. See [Open a Lamina model](/docs/sightline/usage/lamina/opening-a-model).

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
