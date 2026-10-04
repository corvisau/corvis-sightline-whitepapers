The **Portfolio Roll-Up** panel shows one combined coverage and distribution tree
aggregated across every `*.assessment.yaml` binding on a requirement package,
anywhere in the workspace. It is a cross-model view. The roll-up dashboard
inside an open assessment covers only that one binding.

## Opening it

Run **Metron: View Portfolio Roll-Up** from the command palette. It prompts
for a `*.metron.yaml` requirement package, then opens (or reveals) a
read-only **Metron Portfolio** panel for that package. Reopening the command
for the same package reveals the existing panel rather than creating a
second one.

The panel refreshes automatically when a relevant file changes on disk (an
`*.assessment.yaml` binding, or the picked package's own manifest).

## Which bindings are included

The panel includes every `*.assessment.yaml` in the workspace pinned to the
exact same package id and version as the picked package. It folds the pairs
of each into the aggregated tree.

A binding pinned to a different version of the same package is
excluded and listed separately with the reason. This is deliberate. A binding
on an older version can reference requirement ids the current package no
longer has (or has changed). Its pairs cannot safely be folded into the
same tree. To include it, bring the binding up to the current package version
through the existing package-apply or roll-forward flow.

A binding for a different package never appears in either list.

## Exporting

Two commands export the same aggregated data the panel shows:

- **Metron: Export Portfolio CSV**: one row per (requirement × entity)
  pair per included binding. The source model, model id and assessor lead
  each row so rows from different bindings are distinguishable.
- **Metron: Export Portfolio JSON**: the aggregated roll-up tree, plus
  which bindings were included (path + metadata) and which were excluded
  (path + reason).

Both are also available as buttons directly on the open panel, which skip
the package picker. Either way, a save dialog lets you choose where to write the file.

## Coverage on the Tetra diagram

Tetra paints per-entity coverage on its diagram with figures that match this
panel's rules. Orphan pairs are left out, and entities newly in scope count as
draft. See [Coverage overlay](/docs/sightline/usage/tetra/coverage-overlay).
