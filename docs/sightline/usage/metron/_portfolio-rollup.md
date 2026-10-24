The **Portfolio Roll-Up** panel shows one combined coverage and distribution tree
aggregated across every assessment of a requirement package,
anywhere in the workspace. It is a cross-model view, unlike the roll-up dashboard inside an open
assessment, which covers only that one assessment.

## Opening it

Select **Portfolio Roll-Up** under Metron in the Models view, or run
**Metron: View Portfolio Roll-Up** from the command palette. Either way it
prompts for a requirement package, then opens (or reveals) a
read-only **Metron Portfolio** panel for that package. Opening it again
for the same package reveals the existing panel rather than creating a
second one.

The panel refreshes automatically when a relevant assessment, or the picked package, changes on disk.

## Which assessments are included

The panel includes every assessment in the workspace pinned to the
exact same package id and version as the picked package. It folds the pairs
of each into the aggregated tree.

An assessment pinned to a different version of the same package is
deliberately excluded and listed separately with the reason. An assessment
on an older version can reference requirement ids the current package no
longer has (or has changed). To include it, bring the assessment up to the current package version
through the existing package-apply or roll-forward flow.

An assessment for a different package never appears in either list.

## Exporting

Two commands export the same aggregated data the panel shows:

- **Metron: Export Portfolio CSV**: one row per (requirement × entity)
  pair per included assessment. The source model, model id and assessor lead
  each row so rows from different assessments are distinguishable.
- **Metron: Export Portfolio JSON**: the aggregated roll-up tree, plus
  which assessments were included (path + metadata) and which were excluded
  (path + reason).

Both exports are buttons on the open panel. They are also commands in the
command palette, where each asks you to pick one of the workspace's requirement
packages first. Either way, a save dialog lets you choose where to write the
file.

## Coverage on the Tetra diagram

Tetra paints per-entity coverage on its diagram with figures that match this
panel's rules. Orphan pairs are left out, and entities newly in scope count as
draft. See the [Coverage overlay](/docs/sightline/usage/tetra/coverage-overlay) section for
how the diagram displays each coverage state.
