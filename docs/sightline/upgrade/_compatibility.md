This page lists which versions of a model, a template and a command work together. It records the changes that can stop an older file from loading or from rendering.

## Lamina schema versions

A Lamina manifest carries a `schemaVersion`.

| Version | Shape | Status |
|---|---|---|
| 0 | Views can carry an inline `aggregation` field. | Migrated on load |
| 1 | `aggregations[]` holds the aggregations. Views carry `savedFilterId`. Reports carry `filter`, `filterRef` and `aggregation`. | Migrated on load |
| 2 | `dataSlices[]` holds the data slices. Reports use `dataSlice: string`. The manifest has no `views[]`, and views have no `savedFilterId`. | Current |

Lamina migrates a version 0 or version 1 manifest to version 2 in memory when it loads the manifest. See [Open a Lamina model](/docs/sightline/usage/lamina/opening-a-model).

A release can drop the migration for an old version. If the `schemaVersion` of a manifest is below the minimum that the app supports, the app stops. The error asks you to upgrade the manifest first. Upgrade the manifest with a release that still supports its version.

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
