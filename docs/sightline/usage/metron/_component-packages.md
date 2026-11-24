A rule catalogue, an attribute-set catalogue and a scoring-scheme catalogue are
each reusable across requirement packages. Each one lives in its own **component
package**: a folder with its own entry in the Models tree and its own editor.

A requirement package names the component packages it consumes on its
**Packages** tab. It does not copy their content.

## The Four Package Kinds

| Kind | Holds |
|---|---|
| Requirement | Requirements, and the list of component packages it consumes |
| Rules | A catalogue of rules |
| Attribute sets | A catalogue of attribute sets |
| Scheme | A catalogue of scoring schemes |

Every package has a name, an id, a version and an optional description. The name
is the package's one display name: the tree, the editor heading and the package
pickers all show it. A rule package also has a domain.

## Create a Component Package

1. Select **Sightline** in the activity bar, then find the folder in the **Models** tree that should hold the package.
2. Select **New…** on the folder and choose the kind you want.
3. Enter a name and an optional description. A rule package also asks for its **domain**, which is the app that runs its rules: `metron` or `tetra`. The form selects `metron`. The form shows the folder as the location.

Metron creates the package with one starter entry: a rule, an attribute set or a
scheme. Select the package in the tree to open its editor.

Every component-package editor has the same three tabs. The first authors the
catalogue. **Package meta** edits the package name, version and description. **References**
shows which requirement packages consume this one. On a rule package, **References**
also lists the Tetra models that use it. On a rule package, **Package meta** also has a
**Domain** select. Changing it changes which app runs the
package's rules the next time that app runs them; the rules themselves do not
change.

A rule package and a scheme package author in a full-width grid. Select **+ Add**
to create an entry in the props drawer. Click a row to open it in the same
drawer. Enter in a focused cell and a double-click also open it. Right-click a
row for a menu that starts with **Open**, followed by **Delete**. A new entry
and a deletion wait in the save bar until you select **Save all**. An
attribute-set package shows its sets and attributes as a tree, described in the
next section.

A rule package that Tetra authored opens and saves here like any other. A rule keeps
every field it carries when you edit another field of it. The rule drawer shows every field of a rule, the same
drawer that Tetra uses. A requirement can run only a rule that has a program over the model. A
requirement that names a manual rule, or a rule that reads records,
shows an error for that reference and pairs no entity.

The starter attribute set carries a typed attribute. The type is what makes an
attribute assessable, so a set with untyped attributes attaches correctly and
then shows nothing in the Assess drawer.

The starter scheme carries one outcome that maps to `yes` and one that maps to
`no`. It defines no statuses of its own, so the package inherits the three
built-in statuses.

## Edit an Attribute-Set Package

The attribute-set editor shows each set as a row with its attributes beneath it,
in the order the file lists them. Select the arrow on a set to collapse or
expand it.

A set row shows its **id**, its **name**, how many attributes it holds, how many
requirements attach it, and its links. An attribute row shows its name under
**id**, its label under **name**, and its **type**, **owner**, **value** and
**description**.

Click a set or an attribute to open it in the props drawer. Enter in a focused
cell and a double-click also open it. The arrow on a set row only collapses or
expands the set. Right-click a row to open its menu.

| Row | Menu item | Effect |
|---|---|---|
| Set | **Open** | Opens the set in the props drawer. |
| Set | **Add attribute** | Opens a form for a new attribute. The attribute goes last in the set. |
| Set | **Duplicate** | Adds a copy of the set and its attributes, with the id `<id>-copy`. |
| Set | **Delete** | Asks for confirmation, then removes the set and its attributes. |
| Attribute | **Open** | Opens the attribute in the props drawer. |
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

## Consume a Component Package

Open the requirement package and select the **Packages** tab. The tab lists
every package this one consumes, with the kind, the pinned version and what
each contributes.

1. Select **Add package**.
2. Pick a package from the workspace, or name one not yet created by its kind
   and id.

The **Add a component package** dialog lists the packages in the workspace by
kind. To add a package that does not exist yet, use **Name one not yet created**.
Choose the **Package kind**, enter the **Package id** and select **Add by id**.

Each entry names a package by its id, never by a path. Metron scans the
workspace for a package of that kind and id.

**Metron: New Requirement Package** creates a package with an empty Packages tab.
Add each rule, attribute set and scheme package that the new requirement package
uses. The new package opens before you add any of them: it uses a
[built-in scheme](#the-built-in-scheme). Its starter requirement names the rule
`all-zones`. Metron reports an error for that reference until a rule package
on the Packages tab supplies the rule.

The **status** column says what each entry contributes, or why it contributes
nothing:

| Status | Meaning |
|---|---|
| `3 rules` | The package resolves and contributes that many entries |
| `contributes nothing` | The package resolves, and every id it supplies is shadowed by a local one, or it is empty |
| `not installed` | No package of that kind and id is in the workspace |
| `version 2 not installed` | The entry pins a version the installed package does not carry |
| `duplicate id` | Two packages share that kind and id, so the reference is ambiguous. The new-package forms refuse an id that another model already uses. Packages made before that check can still share an id across kinds |

### Column Visibility

The Packages tab has its own **`[Edit columns]`** toolbar button to show, hide and reorder columns. Only **kind** and
**id** are manageable. The **pin** and **status** columns and the row's
**Remove** action carry live controls and stay visible. They have no drag handle and stay after the **kind** and **id** columns.

Drag the handle at the left of the **kind** or **id** header to swap them. You can also focus the handle and use Space and the arrow keys. The picker shows the same order.

Your choice is personal and local.
Dragging a column's resize handle saves its width the same way.

### Remove a Package

Select **Remove** on the entry's row. A removal that breaks a reference asks
first and names what breaks.

⛔ One removal is refused. A scheme package supplying the scheme that the **Scheme**
picker selects cannot be dropped: the package would fail to load and would not open
again. Select another scheme on the **Package meta** tab first.

The referenced entries then appear in the requirement package's own tab for that
kind. Referenced rules are offered in the requirement drawer's rule pickers for
the scope rule and the assessment rule. Referenced attribute sets are offered in the
drawer's attribute-set picker and attach exactly as a local set does.

### Rules and Attribute Sets in a Requirement Package

A requirement package holds no rules or attribute sets of its own. Its **Rules**
and **Attribute sets** tabs are read-only lists of what the packages on the
Packages tab contribute. Neither tab has an add button, a drawer or a delete. Edit an entry
in the component package that owns it.

The **source** column names the package that supplies each entry. Right-click an
entry and select **Open package** to open that package's editor.

A notice above each tab says where its entries are authored. The host also
refuses any rule or attribute-set write against a requirement package,
whatever sends it.

### Legacy Files in a Requirement Package

Metron does not load rules, attribute sets or schemes that sit in a requirement
package's own folder. Such a file contributes nothing, and Metron raises one
warning for each. See [Metron fragment migration](/docs/sightline/upgrade/metron-fragment-migration)
to move them into component packages.

## Select a Scoring Scheme

A rule or an attribute set is consumed by a requirement. A scheme is consumed by
the package: the package is assessed against one scheme, and every requirement in
it uses that scheme.

Metron builds the package's scheme catalogue from every scheme package on the
Packages tab. The package selects one scheme from that catalogue. Adding several
scheme packages widens the choice; it never merges two schemes into one.

1. Open the requirement package and select the **Package meta** tab.
2. Open the **Scheme** picker. It offers every scheme in the catalogue.
3. Select the scheme.

If the selected scheme is no longer in the catalogue, the package fails to load.
The error names the schemes that are available.

### The Built-in Scheme

A new requirement package opens on a built-in scheme with one outcome that maps to
`yes`, **Pass**, and one that maps to `no`, **Fail**. The built-in scheme defines no
statuses of its own, so the package inherits the three built-in statuses.

Metron uses the built-in scheme only while the Packages tab names no scheme
package and the package holds no scheme of its own.

Add a scheme package on the Packages tab to replace the built-in scheme. The
built-in scheme then leaves the catalogue. Select a scheme from the new package in
the **Scheme** picker on the **Package meta** tab in the same change. A scheme
package that supplies a scheme with the id `default` needs no new selection.

A scheme package that is missing, pinned to a version that does not exist, or
invalid still fails the load. A selected scheme other than the built-in one also
fails the load when no scheme package supplies it.

### Scoring Scheme of a Requirement Package

The **Scoring scheme** tab is read-only. Edit a scheme in the scheme package that
owns it.

A banner on the tab names the scheme package that supplies the scheme. Select
**Open** to edit it there.

### Pin a Version, or Float

A pinned entry uses that exact package version. A floating entry resolves to the
single installed package of that id.

Tick **pin** on the Packages tab to pin an entry to the version installed now.
Untick it to float again.

Pin a package when an assessment must stay stable against the rules it was
assessed with. Leave the entry floating while the catalogue is still moving.

Two versions of one package can coexist in a workspace. A floating entry
cannot choose between them: Metron raises a warning naming both packages, and the
entry contributes nothing until you pin it.

### A New Version of a Package

Select **New version** on the **Packages** tab of a requirement package to create the next version.
Metron copies the package into a sibling directory, for example `aescsf-v2` beside `aescsf-v1`, or `aescsf-v2.sightline` beside `aescsf.sightline` for a package in its own folder,
and raises the package version to the next integer. A trailing `-vN` in the directory name and the
package name follows the new version.

The button is unavailable while the package has unsaved changes. Save them first, because Metron
copies the saved files. A message names the new directory and lists each warning, for example a
report template that lies outside the package.

Metron refuses, and writes nothing, in three cases:

- The new directory already exists and holds files.
- The package folder holds a nested component package.
- Another package already has the new version.

The message says which case applies.

The copy holds the package and the files it loads. It holds no assessment and no finding. An
assessment stays pinned to the version it was made against, and a new binding is the way to assess
version 2.

A rules, attribute-set or scheme package has no **New version** button. After such a package gains
a second version, pin every entry that names it.

### Column Visibility

This section covers the Schemes grid in a scheme package's own editor (see
[Create a component package](#create-a-component-package)). The read-only
**Scoring scheme** tab discussed above is separate.

The Schemes toolbar has its own **`[Edit columns]`** button to show, hide and reorder **id**, **outcomes**,
**statuses** and **used by**. Drag the handle at the left of a column header to move that column, or focus the handle and use Space and the arrow keys. The picker shows the same order. Your choice is personal and local. Dragging a column's resize handle saves
its width the same way.

## Leaving an Editor with Changes Pending

Metron asks before an editor swaps out from under your pending changes.

**Opening another package.** **Open package**, from a referenced entry or
from a row in the **References** tab, replaces the editor you are in. With changes
pending it asks first: **Discard** drops them and opens the other package,
**Save** writes them and then opens it, and **Cancel** stays put.

**Opening a row over a new entry.** A new entry you have typed into is not yet
staged. Before a click opens another row, Metron asks whether to discard the new
entry. The **Open** and **Add attribute** menu items ask too. **Discard** drops
the new entry and opens the row. **Cancel** keeps the new entry open. A new
entry with nothing typed into it closes without asking. The requirement editor
does the same for a new requirement.

**Closing the panel.** A package panel with pending changes is marked dirty, so
VS Code shows its own unsaved-changes prompt on close. A refused save keeps the
panel open with your changes intact. A panel opened from the command palette has
no document behind it and so gets no prompt, so save before closing that one.

⛔ **Moving between tabs never asks.** The three tabs of an editor share one set
of pending changes. Your work stays where it was when you switch to
**Package meta** or **References** and back.

**Unsaved rows are tinted.** A row you have created, or edited, since the last
save carries a yellow tint. It also shows a bar on its left edge in the grid.
The tint stays until you save or discard the change, or undo it.

## Undo, Redo and the Journal

**Undo** and **Redo** sit beside the pool save bar in every package editor.
The bar sits at the bottom of the window and shows on every tab of the editor.
A change staged on one tab therefore stays in reach from the others.
They step back and forward through the changes you have staged this session,
one at a time. Undo the only staged edit and the controls stay on screen so
Redo is still reachable, even though nothing is pending any more.

When you stage a new change after an undo, Redo loses whatever it would have restored.

Every Metron editor has a **Journal**: the requirement library, each package
editor and the assessment editor. It is a section of the Sightline
sidebar, below Model Tree, and it starts collapsed. It lists every staged change
in the order you made it, one row per change with three columns: the **Entity**,
the **Operation** and the new **Value**. A created or deleted entry has no value. A long
cell is cut short, and hovering over it shows the whole text. An undone entry stays listed,
struck through, so you can see what you stepped back past. The journal has its own **Undo** and **Redo** buttons, and
they act on the package or assessment you are looking at. The journal clears when you commit.

Undo reaches only what is staged. To recover a change you have already
saved, use git.

## Precedence

A requirement package takes its rules, attribute sets and schemes only from the
packages on the Packages tab.

An id defined in two of those packages resolves to the one in the package added
later. Metron raises a warning naming both the winner and the definition it
shadows.

### Two Different Orderings

⛔ Catalogue precedence and the order of a requirement's attribute sets are separate axes.

| Question | Decided by |
|---|---|
| Which attribute set does the id `mil-levels` resolve to? | Catalogue precedence: the package added later wins |
| A requirement attaches `mil-levels` and `governance`. Which wins on an attribute both define? | The order of the requirement's attribute sets: the later set |

The first picks the set. The second folds the sets a requirement attached.

## Broken Package References

An unresolved Packages entry never fails a load. Metron raises a warning and the
entry contributes nothing:

| Situation | Result |
|---|---|
| No package has that kind and id | Warning naming the entry |
| Two packages claim the id and the entry does not pin a version | Warning naming both files |
| The referenced package has a malformed fragment | Warning naming the package and the parse error |

A requirement that named a rule from the failed entry then reports its own
unresolved-reference diagnostic.

## Reference Counts

A component package's **References** tab counts how often each entry is used,
across every requirement package that names this one. What counts as a
use follows the package kind:

| Kind | Counts |
|---|---|
| Rules | Requirements that use the rule as their scope rule or assessment rule |
| Attribute sets | Requirements that attach the set |
| Scheme | Packages that select the scheme, at one per package |

⛔ Read the count here, not in a requirement package's own tab. That tab counts
requirements in the package you have open. For an entry shared across three
packages it reports only the references in one of them.

An entry that nothing references shows `0` in the References tab, counted across
the whole workspace.

### Tetra Models, Records, Problems and Cause Links

A Tetra model runs every rule of each rule package in its `uses:` list. On a rule
package, these columns add what the Tetra models that list it contribute:

| Column | Where | What it shows |
|---|---|---|
| **used by** | Rules grid | The requirement references to the rule, plus one for each Tetra model that lists the package |
| **records** | Rules grid | The open records of the rule across those Tetra models. A record is open when the rule still emits it and the rule does not hide it |
| **problems** | Rules grid | The problems that the last run of each Tetra model reported for the rule, each starting with the model id |
| **kind** | Consumed by | `requirement package` or `Tetra model` |
| **requirements** | References by entry | The requirement references to the entry |
| **cause links** | References by entry | The Lamina causes that link an open record of the rule. A cause that links the same record twice counts once |

Select a Tetra model in **Consumed by** to open its diagram. Select a requirement
package to open that package.

The counts of records, problems and cause links load after the editor opens, and
their cells stay empty until then. A rule with no open record shows `0`. A run of
the rules in Tetra saves its problems for each package. A dry run, and a **Test
rule** run, save nothing. A Lamina model that still holds an `actor`, `vector` or
`exposure` on a cause cannot be read for cause links. The References tab names it and
leaves its causes out of the count.

The **own links** column of the Rules grid summarises the links that the author of
the rule attached to it. It does not show the records that link to the rule, and it
is hidden until you choose it in the **Columns** picker.

Deleting an entry is not blocked on its reference count. A requirement package
keeps its Packages entry. A requirement that named a deleted rule reports an
unresolved-reference diagnostic on its next load. One that attached a deleted
attribute set inherits nothing from it, with a warning.

⛔ A deleted scheme is the exception. A package that selects it
fails to load until you select another scheme on the **Package meta** tab.

## Column Visibility

The References tab has two grids, each with its own **`[Edit columns]`**
picker. **Consumed by** manages its columns: `package`, `name`, `pinned to` and
`path`, and on a rule package `kind` as well. Its picker appears only
once the workspace scan finishes. **References by entry** manages `entry`,
`title` and `requirements`, and on a rule package `cause links` as well. Its picker
is always shown.
A resized column's width persists per grid too. Each grid also reorders when you drag the handle at the left of a column header. Space and the arrow keys on the handle do the same. A grid's picker shows that grid's order.

## Related Pages

- [Rules](/docs/sightline/usage/metron/rules): what a Rule is, and how scope and assessment use one.
- [Attributes](/docs/sightline/usage/metron/attributes): the attribute model AttributeSets carry, and
  the four layers that fold them.
