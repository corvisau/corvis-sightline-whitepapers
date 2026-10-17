**"How does a requirement decide what it applies to, and how does it get
assessed automatically?"** Both questions route through the same catalog:
a **Rule**.

## One Rule catalog, two reference points

A Rule is `{ ruleId, title, program }`, a named Bonsai set-pipeline program,
stored in `rules/**/*.rules.yaml` fragment files. You author a rule in a **rule
package**, and a requirement package uses it by naming that package in its
`uses:` list (see [Component packages](/docs/sightline/usage/metron/component-packages)). A requirement
package holds no rules of its own: it takes them only from the rule packages it
names. A requirement package created from the Models tree with **Metron: New
Package** names no rule package. Create or import one, and add it to `uses:`. The
same catalog is referenced from two places on a **Requirement**:

- **`scopeRuleRef`** (mandatory): which entities this requirement applies
  to.
- **`assessment.ruleRef`** (only when `assessment.method` is `'rule'`):
  which Rule can auto-suggest an outcome for each in-scope entity.

Both fields are plain Rule references, picked from the same `<select>`.
If the package has no rules yet, the
picker asks you to name a rule package in the manifest's `uses:` list.

## Authoring a Rule

Open the rule package in its own editor. Its **Rules** tab is a full-width grid,
and every create and edit happens in the rule drawer, the slide-in panel. The
Rules panel of Tetra uses the same drawer.

1. Select **+ Add rule**. The drawer opens as **New rule**.
2. Enter a **Rule id** and a **Title**.
3. Author the program, then select **Save**.

The rule waits in the save bar until you select **Save all**. To edit a rule,
right-click its row and select **Edit**. The same menu carries **Delete**.
Left-clicking a row does nothing.

⚠️ **A rule's id is fixed once created.** It is what `scopeRuleRef` and
`assessment.ruleRef` point at, and neither is rewritten on rename, so the drawer
shows it read-only. To change an id, delete the rule and add it again. The drawer
refuses an id that is empty, holds a space or a `/`, or that the package already
holds.

The drawer shows every field of a rule: the title, the description, the links,
the mode, the hide setting, the key parts and data fields, the input and the
rules it depends on. A requirement runs only a rule whose mode is `generator` and
whose input is the model, so a rule you switch to `manual` or `diagnostic`
cannot scope or assess a requirement.

The drawer shows the program as read-only text, one statement on each line, with
a Copy button. A line above the program reads **(no query)**, **1 statement** or
the number of statements. Select **Edit query** to open the shared query builder in a
dialog, the same pipeline editor Requirements and the Worklist use for their own
filters. Select **Apply** to return the edited program to the drawer, or
**Cancel** to keep the program as it was. Escape closes only the dialog, and the
drawer stays open. Applying marks the rule as changed, and it is written with
**Save all** like every other edit.

On a rule with no program, for example a manual rule that you switch to `generator`,
**Edit query** opens the builder on one starter statement: a `where` step that
ends in `emitRecord`. Nothing is written until you select **Apply**, so
**Cancel** leaves the rule without a program.

A rule written without a `mode` that ends in `save` or `add` onto `result` is a
`transform`. Scope rules are transforms, and a Requirement runs one as it runs a
`generator`. A rule with an explicit `mode: generator` must end in `emitRecord`, and
the editor reports the call it ends in when it does not. Change the mode to
`transform` to keep a scope rule that ends in `add()`.

The query builder's relation type checkboxes express kind-narrowing directly in
the program. They read and write an `item.kind |> oneOf(...)` line in place, as
Tetra's data slice edit dialog does for its own relation-type filter. Beyond
that one convenience, a Rule's program is an ordinary Bonsai pipeline: any
`where`/`save`/`add`/`remove`/`emit` combination is valid. `item.kind` is the entity kind (`zone`, `device`, `channel`, and so on). `item.type` is the entity's own domain type, where it has one; see [Data-slice filtering](/docs/sightline/usage/tetra/data-slice-filtering#itemkind-and-itemtype). A **Show code**
button in the dialog's query builder reveals the exact program as
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

- The **source** column names the rule package each rule comes from.
- To change a rule, right-click it and select **Open package**. The rule
  package opens in its own editor. The [Component packages](/docs/sightline/usage/metron/component-packages)
  page describes rule packages.
- The **used by** column counts requirements in this package only. The rule
  package's **References** tab gives the count across the workspace.

The host refuses any rule write against a requirement package, whatever sends
it.

## Column visibility

The Rules grid renders in two places: a rule package's own editor, and a
requirement package's read-only **Rules** tab. It is the grid of the Rules panel
of Tetra. Each place has its own **Columns** button in the grid header to show,
hide and reorder columns, and the header also carries **Filter** and **Group**:

- The rule package's own editor starts with **rule**, **title**, **mode**,
  **used by** and **links**. The requirement package's Rules tab starts with
  **package**, **rule**, **title**, **used by** and **links**. **package** names
  the rule package each rule comes from.
- The picker also offers **input**, **depends on**, **hide**, **severity**,
  **records** and **problems**.
- Drag the handle at the left of a column header to move that column. To use the keyboard, focus the handle, press Space, press the left or right arrow key, then press Space again. The picker shows the same order, and a drag never sorts the column.
- The choice is personal and local, and each place keeps its own.
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
