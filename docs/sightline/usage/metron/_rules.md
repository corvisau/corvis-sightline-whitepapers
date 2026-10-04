**"How does a requirement decide what it applies to, and how does it get
assessed automatically?"** Both questions route through the same catalog:
a **Rule**.

## One Rule catalog, two reference points

A Rule is `{ ruleId, title, program }`, a named Bonsai set-pipeline program,
stored in `rules/**/*.rules.yaml` fragment files. You author a rule in a **rule
package**, and a requirement package uses it by naming that package in its
`uses:` list (see [Component packages](/docs/sightline/usage/metron/component-packages)). A requirement
package's own `rules/` fragments load, but the editor does not change
them. A requirement package created from the Metron group of the Models tree arrives with a rule
package of its own. The same catalog is referenced from two places on a
**Requirement**:

- **`scopeRuleRef`** (mandatory): which entities this requirement applies
  to.
- **`assessment.ruleRef`** (only when `assessment.method` is `'rule'`):
  which Rule can auto-suggest an outcome for each in-scope entity.

Both fields are plain Rule references, picked from the same `<select>`.
If the package has no rules yet, the
picker asks you to name a rule package in the manifest's `uses:` list.

## Authoring a Rule

Open the rule package in its own editor. Its **Rules** tab is a full-width grid,
and every create and edit happens in the **props drawer**, the slide-in panel.

1. Select **+ Add rule**. The drawer opens as a create surface.
2. Enter an ID and a label.
3. Author the program, then select **Create**.

To edit a rule, right-click its row and select **Open**. The same menu carries
**Delete**. Left-clicking a row does nothing.

⚠️ **A rule's id is fixed once created.** It is what `scopeRuleRef` and
`assessment.ruleRef` point at, and neither is rewritten on rename, so the drawer
shows it disabled. To change an id, delete the rule and add it again.

The drawer carries an id/title form plus the shared
query builder, the same pipeline editor
Requirements and the Worklist use for their own filters. The query builder's relation type
checkboxes express kind-narrowing directly in the program. They read and write an `item.kind |> oneOf(...)` line in place, as
Tetra's data slice edit dialog does for its own relation-type filter. Beyond
that one convenience, a Rule's program is an ordinary Bonsai pipeline: any
`where`/`save`/`add`/`remove`/`emit` combination is valid. `item.kind` is the entity kind (`zone`, `device`, `channel`, and so on). `item.type` is the entity's own domain type, where it has one; see [Data-slice filtering](/docs/sightline/usage/tetra/data-slice-filtering#itemkind-and-itemtype). A **Show code**
button in the query builder's controls reveals the exact program as
read-only text with a Copy button.

The query builder's **Templates** tab offers the rule package's own query
templates. They live in the `queryTemplates` list of the rule package's
manifest, and a saved template goes back there.

Nothing blocks a delete. A requirement that references a deleted rule keeps the
reference, and reports an unresolved-reference diagnostic on its next load.
Read the rule package's **References** tab before you delete: its count covers
every requirement package in the workspace.

## Rules in a requirement package

A requirement package's **Rules** tab lists every rule the package can use, and
changes none of them. It has no **+ Add rule** button and no drawer, and it offers no delete.

- The **source** column names the rule package each rule comes from. A rule
  from the package's own `rules/` folder reads `this package`.
- To change a rule from a rule package, right-click it and select **Open
  package**. The rule package opens in its own editor.
- A rule from the package's own folder has no action. To edit it, move it into
  a rule package. The [Component packages](/docs/sightline/usage/metron/component-packages) page
  describes rule packages.
- The **used by** column counts requirements in this package only. The rule
  package's **References** tab gives the count across the workspace.

The host refuses any rule write against a requirement package, whatever sends
it.

## Column visibility

The Rules grid renders in two places: a rule package's own editor, and a
requirement package's read-only **Rules** tab. Each has its own
**`[Edit columns]`** toolbar button to
show, hide and reorder columns:

- The rule package's own editor never lists **source** in its picker. A rule
  package holds only its own rules, so the column would always read `this
  package`. The requirement package's Rules tab does list it, since its grid
  mixes rules from more than one source.
- Drag the handle at the left of a column header to move that column. To use the keyboard, focus the handle, press Space, press the left or right arrow key, then press Space again. The picker shows the same order, and a drag never sorts the column. The handle works in both hosts.
- The choice is personal and local. Hiding a column in either host is
  reflected the next time either mounts.
  A column toggled in the rule package's own editor never changes a
  **source** visibility choice made in the requirement package's Rules tab.
- A column's width saves the same way after the reader drags its resize
  handle.

## Scope execution: the whole program, over the whole model

When a requirement's scope is resolved,
the referenced Rule's entire program runs over every entity in the
bound tetra model. No separate kind pre-filter applies before the
program runs. The program's `.result` register (populated by `save`/`add`/
`remove` tees) becomes the in-scope entity set. A rule used for scope does
not need to `emit(...)` anything. `emit` produces a new value (an
outcome id), which is separate from scope membership.
A scope rule that also ends in `emit(...)` still resolves scope
from its `.result`, not its `.findings`. The two registers are independent
outputs of the same run.

## Assessment execution: `.findings`, matched against the scheme

When a requirement's `assessment.method` is `'rule'`, the referenced Rule is
expected to `emit(...)` an outcome id for each entity it wants to assess.
Unlike scope, assessment reads the program's `.findings` register (populated
by `emit`), not `.result`. For each in-scope entity:

- If the rule emits a value that matches one of the assessment scheme's
  defined outcome ids, that pair gets that outcome as a **draft** suggestion.
  The outcome is never auto-`complete`, because the human always owns the completion boundary.
- If the rule emits a value that does not match any defined outcome id,
  the pair's outcome resolves to a reserved "unrecognized outcome" sentinel.
  The Outcome picker shows the message "Rule
  produced an unrecognized outcome - pick one below" inline, with the normal
  outcome choices, so a human can resolve it.
- If the rule emits nothing for an entity, that pair stays
  unassessed (pending). This is the same state as a fresh `manual` pair.

## Rule catalog vs. Requirement scope/assessment - quick reference

| Concept | Where it lives | What it does |
|---|---|---|
| `Rule.program` | `rules/**/*.rules.yaml` | A Bonsai pipeline; `.result` (save/add/remove) = a candidate set, `.findings` (emit) = tagged values |
| `Requirement.scopeRuleRef` | `*.req.yaml` | Points at a Rule; its `.result` over the whole model = this requirement's in-scope entities |
| `Requirement.assessment.ruleRef` | `*.req.yaml` | Points at a Rule (only when `assessment.method: 'rule'`); its `.findings` = suggested outcomes, matched against the scheme |
