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

Every manifest carries the same `meta` block: `id`, `name`, `version` and an
optional `description`. `name` is required and is the package's one display name:
the tree, the editor heading and the package pickers all show it. A rule package
also carries `domain`. A manifest declares its kind in a top-level
`kind` field. A requirement-package manifest may omit `kind`.

⛔ The filename does not decide the kind. A glob `*` spans dots, so
`**/*.metron.yaml` matches all four manifests. Metron reads the `kind` field
everywhere: in the Models tree, in the editor it opens, and in the resolver.

## Create a component package

1. Select **Sightline** in the activity bar, then find the folder in the **Models** tree that should hold the package.
2. Select **New…** on the folder and choose the kind you want.
3. Enter a name and an optional description. A rule package also asks for its **domain**, which is the app that runs its rules: `metron` or `tetra`. The form selects `metron`. The form shows the folder as the location.

The scaffold writes two files beside each other, with no subfolder: the manifest
and one content file named after the package, holding a starter entry.

| Kind | Content file | Starter entry | `files:` in the manifest |
|---|---|---|---|
| Rules | `<name>.rules.yaml` | A rule | `<name>.rules.yaml` |
| Attribute sets | `<name>.attributes.yaml` | An attribute set | `<name>.attributes.yaml` |
| Scheme | `<name>.scheme.yaml` | A scheme | `*.scheme.yaml` |

Select the package in the tree to open its editor.

### The `files:` key

Every component-package manifest declares `files:`, a list of the content files
or glob patterns it holds, relative to the manifest. Metron refuses a manifest
without the key and names it in the error. An empty list, `files: []`, declares a
package with no content.

A scheme package uses a pattern because each scheme is a file of its own, and a
scheme added later is found through the pattern. A package authored before this
key was required needs it added. A migration tool converts a model in the earlier format. Ask Corvis for the tool that matches your release.
The tool writes the previous default pattern into each manifest that lacks the key,
and leaves every other manifest unchanged.

Every component-package editor has the same three tabs. The first authors the
catalogue. **Package meta** edits the `meta` block. **References** shows which
requirement packages consume this one. On a rule package, **Package meta** also
has a **Domain** select. Changing it changes which app runs the package's rules
the next time that app runs them; the rules themselves do not change.

A rule package authors in a full-width grid. Select **+ Add** to create an
entry in the props drawer. Right-click a row to **Open** it in the same drawer,
or to **Delete** it. A new entry and a deletion wait in the save bar until you
select **Save all**. An attribute-set package shows its sets and attributes as a
tree, described in the next section. A scheme package edits the selected scheme
in a panel beside its grid.

A rule package that Tetra authored opens and saves here like any other. A rule keeps
every field it carries, such as `mode`, `hide`, `keyParts` and `dataFields`, when
you edit another field of it. The rule drawer shows every field of a rule, the same
drawer that Tetra uses. A requirement can run only a rule that has a program over the model. A
requirement that names a manual rule, or a rule that reads records,
shows an error for that reference and pairs no entity.

The starter attribute set carries a typed attribute. The type is what makes an
attribute assessable, so a set with untyped attributes attaches correctly and
then shows nothing in the Assess drawer.

The starter scheme carries one `yes` outcome and one `no` outcome. It declares no
`statuses`, so the package inherits the three built-in statuses.

## Edit an attribute-set package

The attribute-set editor shows each set as a row with its attributes beneath it,
in the order the file lists them. Select the arrow on a set to collapse or
expand it.

A set row shows its **id**, its **name**, how many attributes it holds, how many
requirements attach it, and its links. An attribute row shows its name under
**id**, its label under **name**, and its **type**, **owner**, **value** and
**description**.

Right-click a row to open its menu.

| Row | Menu item | Effect |
|---|---|---|
| Set | **Edit** | Opens the set in the props drawer. |
| Set | **Add attribute** | Opens a form for a new attribute. The attribute goes last in the set. |
| Set | **Duplicate** | Adds a copy of the set and its attributes, with the id `<id>-copy`. |
| Set | **Delete** | Asks for confirmation, then removes the set and its attributes. |
| Attribute | **Edit** | Opens the attribute in the props drawer. |
| Attribute | **Duplicate** | Adds a copy directly after the attribute, named `<name>-copy`. |
| Attribute | **Remove** | Removes the attribute. |

Every change waits in the save bar until you select **Save all**. **Discard all**
returns every row to its saved state. A row with an unsaved change is
highlighted.

The name of an attribute is fixed once the attribute exists. To change a name,
add an attribute with the new name, then remove the original.

The filter above the tree matches the id and name of a set, and the name, label,
value, type, owner and description of an attribute. A set that holds a matching
attribute stays in view, with only the matching attributes beneath it.

The create form for a new set can list its first attributes. The props drawer of
an existing set shows how many attributes it holds. Edit them in the tree.

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
  name: AESCSF v1
  version: 1
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

**Metron: New Package** writes this list empty. Add each rule, attribute set and
scheme package that the new requirement package uses.

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

A requirement package holds no rules or attribute sets of its own. Its **Rules**
and **Attribute sets** tabs are read-only lists of what the packages in `uses:`
contribute. Neither tab has an add button, a drawer or a delete. Edit an entry
in the component package that owns it.

The **source** column names the package that supplies each entry. Right-click an
entry and select **Open package** to open that package's editor.

A notice above each tab says where its entries are authored. The host also
refuses any rule or attribute-set write against a requirement package,
whatever sends it.

### A package that still holds its own fragments

Metron does not load a `rules/`, `attributes/` or `*.scheme.yaml` file that sits
in a requirement package's own folder. Such a file contributes nothing.

Metron raises one warning for each such file. The warning names the file. A
second migration tool moves it into a component package.

The tool creates a rule package, an attribute-set package and a scheme
package beside the requirement package, moves each fragment into the matching
package, and adds the packages to `uses:`. It offers a dry run that shows what
it would do first.

⛔ A package whose `schemeRef` names a scheme in its own folder does not open
until you move the scheme into a scheme package. A package with no scheme has no outcomes to show.

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
`uses:`. `schemeRef` selects one entry from that catalogue. Naming several
scheme packages widens the choice; it never merges two schemes into one.

Select the scheme on the **Package meta** tab. The picker offers every id in the
catalogue.

A `schemeRef` that matches no id fails the load. The error names the ids
available and the line to write.

### A requirement package does not author its scheme

The **Scoring scheme** tab is read-only. Edit a scheme in the scheme package that
owns it.

A banner on the tab names the scheme package that supplies the scheme. Select
**Open** to edit it there.

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

**Unsaved rows are tinted.** A row you have created, or edited, since the last
save carries a yellow tint and a bar on its left edge in the grid. The tint
stays until you save or discard the change, or undo it.

## Undo, redo and the journal

**Undo** and **Redo** sit beside the pool save bar in every package editor.
The bar sits at the bottom of the window and shows on every tab of the editor,
so a change staged on one tab stays in reach from the others.
They step back and forward through the changes you have staged this session,
one at a time. Undo the only staged edit and the controls stay on screen so
Redo is still reachable, even though nothing is pending any more.

When you stage a new change after an undo, Redo loses whatever it would have restored.

Every Metron editor has a **Journal**: the requirement library, each package
editor and the assessment editor. It is a section of the Sightline
sidebar, below Model Tree, and it starts collapsed. It lists every staged change
in the order you made it. An undone entry stays listed, struck through, so you can see what you
stepped back past. The journal has its own **Undo** and **Redo** buttons, and
they act on the package or assessment you are looking at. The journal clears when you commit.

Undo reaches only what is staged. To recover a change you have already
saved, use git.

## Precedence

A requirement package takes its rules, attribute sets and schemes only from the
packages in `uses:`.

An id defined in two of those packages resolves to the one in the later `uses:`
entry. Metron raises a warning naming both the winner and the definition it
shadows.

### Two different orderings

⛔ Catalogue precedence and `attributeSetRefs` order are separate axes.

| Question | Decided by |
|---|---|
| Which attribute set does the id `mil-levels` resolve to? | Catalogue precedence: the later `uses:` entry wins |
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
picker. **Consumed by** manages its 4 columns (`package`, `name`,
`pinned to`, `path`). Its picker appears only
once the workspace scan finishes. **References by entry** manages its 3
(`entry`, `title`, `references`). Its picker is always shown.
A resized column's width persists per grid too. Each grid also reorders when you drag the handle at the left of a column header. Space and the arrow keys on the handle do the same. A grid's picker shows that grid's order.

## Related pages

- [Rules](/docs/sightline/usage/metron/rules): what a Rule is, and how scope and assessment use one.
- [Attributes](/docs/sightline/usage/metron/attributes): the attribute model AttributeSets carry, and
  the four layers that fold them.
