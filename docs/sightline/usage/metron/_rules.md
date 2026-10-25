A requirement's scope and its assessment both route through the same catalog:
a **Rule**.

## One Rule catalog, two reference points

A Rule has an id, a title and a program, a named Bonsai set-pipeline program.
You author a rule in a **rule package**, and a requirement package uses it by
adding that package on its **Packages** tab (see
[Component packages](/docs/sightline/usage/metron/component-packages)). A requirement
package holds no rules of its own: it takes them only from the rule packages it
names. A requirement package created from the Models tree with **Metron: New
Package** names no rule package. Create or import one, and add it on the
**Packages** tab with **Add package**. The
same catalog is referenced from two places on a **Requirement**:

- **Scope rule** (mandatory): which entities this requirement applies
  to.
- **Assessment rule** (only when **Assessment method** is `rule`):
  which Rule can auto-suggest an outcome for each in-scope entity.

Both fields are plain Rule pickers in the requirement drawer.
If the package has no rules yet, the
picker is empty until you add a rule package on the **Packages** tab.

## Authoring a Rule

Open the rule package in its own editor. Its **Rules** tab is a full-width grid,
and every create and edit happens in the rule drawer, the slide-in panel. The
Rules panel of Tetra uses the same drawer.

1. Select **+ Add rule**. The drawer opens as **New rule**.
2. Enter a **Rule id** and a **Title**. A rule needs a title.
3. Choose a **Mode**. The drawer selects `generator` and gives each mode a one-line explanation.
4. Select **Edit query**. The builder opens on a starter statement for the mode. Select **Apply** to write it as the program, and change it as needed.
5. Select **Save**.

A new rule starts with no program, and **Save** stays disabled until the rule has one.
The line above the program names the call it must end in: `emitRecord(...)` for a
`generator` and `save(result)` for a `transform`. A `manual` rule has no program.

| Mode | What the rule does |
|---|---|
| `generator` | Runs its program over the model, or over other rules' records, and writes one record for each `emitRecord` call. |
| `manual` | Has no program. People add its records by hand. |
| `transform` | Runs its program over the model and returns the rows that a requirement scopes or assesses with. It writes no records. |

The drawer says what is wrong in three places:

- A message under a field names the problem with that field. A program that ends in the wrong call for its mode gets a message that names the fix.
- A line above the buttons says why **Save** and **Test rule** are off. After a test that reports an error, it says that **Save** stays off until the rule changes.
- **Depends on** warns about a rule that the package does not have. The warning does not stop a save, because the rule can be in a package that is not loaded.

A help icon beside each field opens a short explanation. It opens on hover and when you tab to it.

The rule waits in the save bar until you select **Save all**. To edit a rule,
right-click its row and select **Edit**. The same menu carries **Delete**.
Left-clicking a row does nothing.

⚠️ **A rule's id is fixed once created.** It is what the **Scope rule** and
**Assessment rule** pickers point at, and neither is rewritten on rename, so the drawer
shows it read-only. To change an id, delete the rule and add it again. The drawer
refuses an id that is empty, holds a space or a `/`, or that the package already
holds.

The drawer shows the fields of a rule in this order:

1. Rule id and title
2. Mode
3. Program
4. Input and the rules it depends on
5. Hide setting, key parts and data fields
6. Links and description

Key parts, data fields and the rules it depends on are lists of rows. Add a row,
remove a row, or drag a row by its handle to change the order. The Tetra page
[Rules and findings](/docs/sightline/usage/tetra/rules-and-findings) lists the controls.

A mode shows only the fields it uses. When you change the mode, a line under the
choice names the values that the new mode drops on **Save**.

A requirement runs only a rule whose mode is `generator` or `transform` and whose
input is the model. A `manual` rule, or a rule that reads records, cannot scope or
assess a requirement.

The drawer shows the program as a list of steps. Each statement is a numbered
row, with a badge that says what it does and a short description of what it
selects. A generator's last step has the **Emit** badge. A comment shows as a
caption, and a disabled statement stays in its place with an **off** badge.
**Show code** shows the program as text, one statement on each line, with a Copy
button. A line above the program reads **(no query)**, **1 statement** or
the number of statements. Select **Edit query** to open the shared query builder in a
dialog, the same pipeline editor Requirements and the Worklist use for their own
filters. Select **Apply** to return the edited program to the drawer, or
**Cancel** to keep the program as it was. Escape closes only the dialog, and the
drawer stays open. Applying marks the rule as changed, and it is written with
**Save all** like every other edit.

On a rule with no program, **Edit query** opens the builder on one starter
statement. A new rule and a manual rule that you switch to `generator` start from
a `where` step that ends in `emitRecord`. A `transform` starts from a `where` step
that ends in `save(result)`. Nothing is written until you select **Apply**, so
**Cancel** leaves the rule without a program.

A rule that ends in `save` or `add` onto `result` is a `transform`. Scope rules
are transforms. A rule with the mode `generator` must end in `emitRecord`, and
the editor reports the call it ends in when it does not. Change the mode to
`transform` to keep a scope rule that ends in `add()`.

The query builder's `includeKinds` and `excludeKinds` steps express kind-narrowing
directly in the program, with a checkbox for every entity kind in the step's
editor, as Tetra's data slice edit dialog does. A scope rule
that narrows by kind lets a requirement match a slice's target kinds: an
`includeKinds("zone")` rule targets zones, and an `excludeKinds("information")`
rule targets every kind except information. Beyond those steps, a Rule's program is an ordinary Bonsai pipeline: any
`where`/`set`/`save`/`add`/`remove` combination is valid, and a generator ends in `emitRecord`. `item.kind` is the entity kind (`zone`, `device`, `channel`, and so on). `item.type` is the entity's own domain type, where it has one; see [Data-slice filtering](/docs/sightline/usage/tetra/data-slice-filtering#itemkind-and-itemtype). A **Show code**
button in the dialog's query builder reveals the exact program as
read-only text with a Copy button.

The query builder's **Templates** tab offers the rule package's own query
templates. A template that you save goes back to the rule package.

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
program runs. The program's `result` register (populated by `save`/`add`/
`remove`) becomes the in-scope entity set.

## Assessment execution: rows with an outcome, matched against the scheme

When a requirement's **Assessment method** is `rule`, the referenced Rule is a
`transform`. It ends in `save(result)` or `add()` and returns one row for each
entity it assesses. Each row carries an `outcome` field that names an outcome of
the assessment scheme. The `set` step writes the field:

```
model |> where("item.kind == 'zone' && item.isCrossZoneExposed == true") |> set("outcome", "'mil-0'") |> save(result)
```

A rule with several statements adds each one's rows with `add()`. An entity that
two statements match takes the outcome of the later one.

A generator rule cannot assess a requirement. The requirement package reports
the rule as one a requirement cannot run.

The run writes each outcome into the rule layer of the pair's record. The layer
is rebuilt on every run, so a rule that stops returning an entity removes the
outcome it gave. For each in-scope entity:

- If the row's outcome matches a defined outcome id of the assessment scheme,
  the pair shows that outcome. The assessor still has none of their own.
  The outcome is never auto-`complete`, because the human always owns the completion boundary.
- If the row's outcome matches no defined outcome id, the pair's outcome
  resolves to a reserved "unrecognized outcome" sentinel. The Outcome picker
  shows the message "Rule produced an unrecognized outcome - pick one below"
  inline, with the normal outcome choices. A human can then resolve it.
- If the rule returns no row for an entity, that pair stays unassessed
  (pending). This is the same state as a fresh `manual` pair.

An outcome that the assessor sets takes precedence over the rule's. Suppose the
assessor moves a pair to a `done` status without setting an outcome. The pair
then keeps the rule's outcome as its own, so a later run never changes a
completed pair.

A row with no string `outcome`, or a program that saves a single value, produces
a warning and assesses nothing.

## Rule catalog vs. Requirement scope/assessment - quick reference

| Concept | Where you set it | What it does |
|---|---|---|
| Rule program | The rule drawer of a rule package | A Bonsai pipeline; its `result` register (save/add/remove) holds the rows it returns |
| **Scope rule** | The requirement drawer | Points at a Rule; its `result` over the whole model = this requirement's in-scope entities |
| **Assessment rule** | The requirement drawer, when **Assessment method** is `rule` | Points at a transform Rule; the `outcome` of each row it returns = the suggested outcome, matched against the scheme |
