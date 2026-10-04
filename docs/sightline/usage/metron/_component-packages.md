A rule catalogue, an attribute-set catalogue and a scoring-scheme catalogue are
each reusable across requirement packages. Each one lives in its own **component
package**: a directory with its own manifest, its own entry in the Models tree,
and its own editor.

A requirement package names the component packages it consumes in a `uses:`
list. It does not copy their files.

## The four package kinds

| Kind | Manifest | Holds |
|---|---|---|
| Requirement | `<name>.metron.yaml` | Requirements, and a `uses:` list |
| Rules | `<name>.rules.metron.yaml` | A Rule catalogue |
| Attribute sets | `<name>.attributes.metron.yaml` | An AttributeSet catalogue |
| Scheme | `<name>.scheme.metron.yaml` | An AssessmentScheme catalogue |

Every manifest carries the same `meta` block: `id`, `name`, `version`, `title`
and an optional `description`. A manifest declares its kind in a top-level
`kind` field. A requirement-package manifest may omit `kind`, which is why every
package authored before this feature still loads.

⛔ The filename does not decide the kind. A glob `*` spans dots, so
`**/*.metron.yaml` matches all four manifests. Metron reads the `kind` field
everywhere: in the Models tree, in the editor it opens, and in the resolver.

## Create a component package

1. Select **Sightline** in the activity bar, then find the **Metron** group of the **Models** tree.
2. Select the **+** action of the group for the kind you want.
3. Enter a name, a location and an optional description.

The scaffold writes the manifest and one starter entry:

- A rule at `rules/general.rules.yaml`
- An attribute set at `attributes/general.attributes.yaml`
- A scheme at `schemes/default.scheme.yaml`

Select the package in the tree to open its editor.

Every component-package editor has the same three tabs. The first authors the
catalogue. **Package meta** edits the `meta` block. **References** shows which
requirement packages consume this one.

A rule package and an attribute-set package author in a full-width grid. Select
**+ Add** to create an entry in the props drawer. Right-click a row to **Open**
it in the same drawer, or to **Delete** it. A new entry and a deletion wait in
the save bar until you select **Save all**. A scheme package edits the selected
scheme in a panel beside its grid.

The starter attribute set carries a typed attribute. The type is what makes an
attribute assessable, so a set with untyped attributes attaches correctly and
then shows nothing in the Assess drawer.

The starter scheme carries one `yes` outcome and one `no` outcome. It declares no
`statuses`, so the package inherits the three built-in statuses.

## Consume a component package

Open the requirement package and select the **Packages** tab. The tab lists
every package this one consumes, with the kind, the pinned version and what
each contributes.

1. Select **Add package**.
2. Pick a package from the workspace, or name one not yet created by its kind
   and id.

The tab edits the manifest's `uses:` list, which reaches disk when you Save:

```yaml
meta:
  id: aescsf
  name: aescsf-v1
  version: 1
  title: AESCSF v1
schemeRef: mil
files:
  - requirements/*.req.yaml
uses:
  - kind: rules
    id: ot-common-rules
    version: 2
  - kind: scheme
    id: ot-schemes
```

Each entry names a package by its logical `id`, never by a path. Metron scans
the workspace for a manifest of that kind and id.

The **+** action of the Metron group writes this list too: a new requirement package
gets a rule package and a scheme package beside it, already named.

The **status** column says what each entry contributes, or why it contributes
nothing:

| Status | Meaning |
|---|---|
| `3 rules` | The package resolves and contributes that many entries |
| `contributes nothing` | The package resolves, and every id it supplies is shadowed by a local one, or it is empty |
| `not installed` | No package of that kind and id is in the workspace |
| `version 2 not installed` | The entry pins a version the installed package does not carry |
| `duplicate id` | Two packages share that kind and id, so the reference is ambiguous |

### Column visibility

The Packages tab has its own **`[Edit columns]`** toolbar button to show, hide and reorder columns. Only **kind** and
**id** are manageable. The **pin** and **status** columns and the row's
**Remove** action carry live controls and stay visible. They have no drag handle and stay after the **kind** and **id** columns.

Drag the handle at the left of the **kind** or **id** header to swap them. You can also focus the handle and use Space and the arrow keys. The picker shows the same order.

Your choice is personal and local.
Dragging a column's resize handle saves its width the same way.

### Remove a package

Select **Remove** on the entry's row. A removal that breaks a reference asks
first and names what breaks.

⛔ One removal is refused. A scheme package supplying the scheme `schemeRef`
selects cannot be dropped: an unresolved `schemeRef` fails the load, so the
package would not open again. Point `schemeRef` at another scheme first.

The referenced entries then appear in the requirement package's own tab for that
kind. Referenced rules are offered in the rule picker for `scopeRuleRef` and
`assessment.ruleRef`. Referenced attribute sets are offered in a requirement's
`attributeSetRefs` picker and attach exactly as a local set does.

### A requirement package does not author rules or attribute sets

A requirement package's **Rules** and **Attribute sets** tabs are read-only
lists. Neither has an add button, a drawer or a delete. Edit an entry in the
component package that owns it.

| Entry | **source** column | Right-click |
|---|---|---|
| From a package in `uses:` | Names the package | **Open package** opens that package's editor |
| From the package's own `rules/` or `attributes/` folder | `this package` | No action. Move the entry into a component package to edit it |

A requirement package created from the Metron group of the Models tree arrives with a rule package
beside it. The second row covers a package authored before that, or one that
still holds its own fragments.

A notice above each tab says where its entries are authored. The host also
refuses any rule or attribute-set write against a requirement package,
whatever sends it.

## Select a scoring scheme

A rule or an attribute set is consumed by a requirement. A scheme is consumed by
the package: `schemeRef` names the one scheme every requirement in the package is
assessed against.

`schemeRef` holds an `AssessmentScheme.id`. It never holds a file path.

```yaml
schemeRef: mil
uses:
  - kind: scheme
    id: ot-schemes
```

Metron builds the package's scheme catalogue from every scheme package in
`uses:`, then folds the package's own `**/*.scheme.yaml` files on top.
`schemeRef` selects one entry from that catalogue. Naming several scheme packages
widens the choice; it never merges two schemes into one.

Select the scheme on the **Package meta** tab. The picker offers every id in the
catalogue.

A `schemeRef` that matches no id fails the load. The error names the ids
available and the line to write.

### A requirement package does not author its scheme

The **Scoring scheme** tab is read-only, whatever the scheme's origin. Edit a
scheme in the scheme package that owns it.

A banner on the tab names where the scheme lives:

| Origin | Banner | Action |
|---|---|---|
| A referenced scheme package | Names the package | Select **Open** to edit it there |
| The package's own `*.scheme.yaml` | Names the file | Open the file to edit it |

A requirement package created from the Metron group of the Models tree arrives with a scheme
package beside it. The second row covers a package that still holds its own
`*.scheme.yaml`.

⛔ Metron discovers a package's own schemes at `**/*.scheme.yaml`. A scheme file
named outside that pattern is never loaded. Rename it. Then set `schemeRef` to
the `id` inside it.

### Pin a version, or float

A `version` field pins the entry to that exact package version. Omit it and the
entry resolves to the single installed package of that id.

Tick **pin** on the Packages tab to pin an entry to the version installed now.
Untick it to float again.

Pin a package when an assessment must stay stable against the rules it was
assessed with. Leave the entry floating while the catalogue is still moving.

Two versions of one package can coexist in a workspace. An unpinned entry
cannot choose between them: Metron raises a warning naming both files, and the
entry contributes nothing until you pin it.

### A new version of a package

Metron has no in-app command that creates version 2 of a package. A new version
lives in a sibling directory, for example `aescsf-v2/` beside `aescsf-v1/`, with
`meta.version` raised to the next integer.

An assessment stays pinned to the version it was made against, and a new binding
is the way to assess version 2. After a rules, attribute-set or scheme package
gains a second version, pin every `uses:` entry that names it.

### Column visibility

This section covers the Schemes grid in a scheme package's own editor (see
[Create a component package](#create-a-component-package)). The read-only
**Scoring scheme** tab discussed above is separate.

The Schemes toolbar has its own **`[Edit columns]`** button to show, hide and reorder **id**, **outcomes**,
**statuses** and **used by**. Drag the handle at the left of a column header to move that column, or focus the handle and use Space and the arrow keys. The picker shows the same order. Your choice is personal and local. Dragging a column's resize handle saves
its width the same way.

## Leaving an editor with changes pending

Metron asks before an editor swaps out from under your pending changes.

**Opening another package.** **Open package**, from a referenced entry or
from a row in the **References** tab, replaces the editor you are in. With changes
pending it asks first: **Discard** drops them and opens the other package,
**Save** writes them and then opens it, and **Cancel** stays put.

**Closing the panel.** A package panel with pending changes is marked dirty, so
VS Code shows its own unsaved-changes prompt on close. A refused save keeps the
panel open with your changes intact. A panel opened from the command palette has
no document behind it and so gets no prompt, so save before closing that one.

⛔ **Moving between tabs never asks.** The three tabs of an editor share one set
of pending changes, so your work stays where it was when you switch to
**Package meta** or **References** and back.

## Undo, redo and the journal

**Undo** and **Redo** sit beside the pool save bar in every package editor.
They step back and forward through the changes you have staged this session,
one at a time. Undo the only staged edit and the controls stay on screen so
Redo is still reachable, even though nothing is pending any more.

When you stage a new change after an undo, Redo loses whatever it would have restored.

The requirement library also has a **Journal**. It is a section of the Sightline
sidebar, below Model Tree, and it starts collapsed. It lists every staged change
in the order you made it. An undone entry stays listed, struck through, so you can see what you
stepped back past. The journal has its own **Undo** and **Redo** buttons, and
they act on the package you are looking at. The journal clears when you commit.

Undo reaches only what is staged. To recover a change you have already
saved, use git.

## Precedence

A requirement package's own fragments still load: `rules/**/*.rules.yaml`,
`attributes/**/*.attributes.yaml` and `**/*.scheme.yaml`. They take precedence
over anything a `uses:` entry contributes.

Resolution order, lowest precedence first:

1. Entries from each package in `uses:`, in `uses:` order.
2. The requirement package's own fragments.

An id defined in two places resolves to the last one in this order. Metron
raises a warning naming both the winner and the definition it shadows.

A local entry with the same id as a referenced one therefore overrides it for
this package alone. Metron does not write local entries, so such an override
exists only where one is written by hand in YAML.

### Two different orderings

⛔ Catalogue precedence and `attributeSetRefs` order are separate axes.

| Question | Decided by |
|---|---|
| Which attribute set does the id `mil-levels` resolve to? | Catalogue precedence: local beats referenced, later `uses:` beats earlier |
| A requirement attaches `mil-levels` and `governance`. Which wins on an attribute both define? | `attributeSetRefs` order: the later ref |

The first picks the set. The second folds the sets a requirement attached.

## What a broken reference does

An unresolved `uses:` entry never fails a load. Metron raises a warning and the
entry contributes nothing:

| Situation | Result |
|---|---|
| No package has that kind and id | Warning naming the entry |
| Two packages claim the id and the entry does not pin a version | Warning naming both files |
| The referenced package has a malformed fragment | Warning naming the package and the parse error |

A requirement that named a rule from the failed entry then reports its own
unresolved-reference diagnostic.

## Reference counts

A component package's **References** tab counts how often each entry is used,
across every requirement package that names this one in `uses:`. What counts as a
use follows the package kind:

| Kind | Counts |
|---|---|
| Rules | Requirements naming the rule in `scopeRuleRef` or `assessment.ruleRef` |
| Attribute sets | Requirements attaching the set in `attributeSetRefs` |
| Scheme | Packages whose `schemeRef` selects the scheme, at one per package |

⛔ Read the count here, not in a requirement package's own tab. That tab counts
requirements in the package you have open. For an entry shared across three
packages it reports only the references in one of them.

An entry that nothing references shows `0` in the References tab, counted across
the whole workspace.

Deleting an entry is not blocked on its reference count. A requirement package
keeps its `uses:` entry. A requirement that named a deleted rule reports an
unresolved-reference diagnostic on its next load. One that attached a deleted
attribute set inherits nothing from it, with a warning.

⛔ A deleted scheme is the exception. A package whose `schemeRef` selects it
fails to load until you re-point `schemeRef`.

## Column visibility

The References tab has two grids, each with its own **`[Edit columns]`**
picker. **Consumed by** manages its 4 columns (`package`, `title`,
`pinned to`, `path`). Its picker appears only
once the workspace scan finishes. **References by entry** manages its 3
(`entry`, `title`, `references`). Its picker is always shown.
A resized column's width persists per grid too. Each grid also reorders when you drag the handle at the left of a column header. Space and the arrow keys on the handle do the same. A grid's picker shows that grid's order.

## Related pages

- [Rules](/docs/sightline/usage/metron/rules): what a Rule is, and how scope and assessment use one.
- [Attributes](/docs/sightline/usage/metron/attributes): the attribute model AttributeSets carry, and
  the four layers that fold them.
