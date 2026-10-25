The Rules panel of Tetra lists the rules of the packages a model uses. It edits a rule and tests it before the rule is saved. It adds, removes and pins the packages the model uses. It also lists the records of the model. A model produces records only from the packages it uses. See [Rule packages](/docs/sightline/usage/tetra/rule-packages).

## Open the Rules panel

Select **Rules** in the Tetra toolbar, or run **Tetra: Edit Rules**. The panel opens for the model of the active diagram. Each model has one panel, and a second request reveals the open one.

The button is disabled in a window that does not allow edits. In the browser, it opens the Rules panel in its own web shell tab. The diagram and its unsaved changes stay open. A Rules tab opened with a reader token has its editing controls disabled.

The panel has three tabs, **Rules**, **Packages** and **Records**.

## The Rules tab

The tab lists one row for each rule of each package the model uses. Two packages that define the same rule id give two rows.

| Column | Content |
|---|---|
| package | The id of the rule package |
| rule | The rule id |
| title | The title of the rule |
| mode | `generator` for a rule with a program that writes records, `manual` for a rule whose records a person creates, `transform` for a rule with a program that creates no record |
| input | `model` or `records`. A manual rule has none, and a transform reads `model`. |
| hide | Whether the records of the rule stay out of the consumer lists, such as the Lamina inbox. A generator with hide set writes diagnostics. |
| records | The number of records the rule has created for this model |
| problems | The message of a problem that names the rule |

The grid starts with the package, rule, title, mode and records columns. Use **Columns** to show the others. The panel remembers your choice.

A package entry that matches no installed package, or more than one, appears above the grid as a problem. The entry adds no rule.

The records column of a diagnostic rule counts the records the rule created. See [Write a diagnostic rule](/docs/sightline/usage/tetra/rule-packages#write-a-diagnostic-rule).

A model that uses no package shows **Add a rule package** above the grid. The button opens the **Packages** tab.

### Columns, Filter and Group

The header of the grid carries the same controls as the Tabular view.

- **Columns** shows, hides and reorders the columns. A column can also move when you drag its header.
- **Filter** narrows the rows. The Basic tab builds conditions on the package, rule, title, mode, input, depends on and hide fields. The Advanced tab edits the same filter as a query.
- **Group** groups the rows by package, mode, input or hide.
- The filter button in a column header lists the values of that column. Clear a value to hide the rows that carry it.

Select a row to open the rule in a drawer at the side of the grid. The Close button and the Escape key close the drawer.

## The Packages tab

The tab edits the list of packages the model uses. It lists one row for each entry.

| Column | Content |
|---|---|
| Package | The id of the rule package, and its name |
| Version | The version the entry uses. Select another version to change it. |
| Rules | The number of rules in the package |
| Records | The number of records the package has stored for this model |
| Status | The version in use, or the reason that the entry adds no rule |

### Add a package

1. Select a package in the **Package** list. The list shows the rule packages installed under the workspace root that the model does not use yet.
2. Select a version when the package has more than one installed version. A package with one installed version follows that version.
3. Select **Add**.

### Change the version of a package

Select a version in the **Version** list of the row. **Any installed version** removes the pin, and the entry then follows the one installed version. The list offers it only while one version of the package is installed.

### Remove a package

Select **Remove** in the row of the package. A package that has stored records asks for confirmation first. The records stay readable, but their rules are unavailable, so a run does not update them. Add the package again to bring its rules back.

### How the tab saves

Each change saves the list at once. The rest of the model definition stays as it was.

The tab does not overwrite a model definition that changed outside the panel after the panel loaded it. The panel reloads the list and shows a message. Repeat the change.

## Edit a rule

The drawer edits one rule. The rule id is fixed once the rule exists.

| Field | Content |
|---|---|
| Title | The title of the rule. A rule needs a title, and the drawer refuses a blank one. |
| Description | A sentence that says what the rule finds |
| Mode | `generator`, `manual` or `transform`. The line under the select says what the chosen mode does. |
| Hide from consumer lists | Whether the records of the rule stay out of consumer lists such as the Lamina inbox. A rule shows its records unless this is set. |
| Input | `model` for a rule that reads the entities of the model, or `records` for a rule that reads the records of other rules. Only a `generator` rule has an input. |
| Depends on | A list with one row for each rule whose records this rule reads or must run after. Write a rule of the same package as `rule-id`, and a rule of another package as `package-id/rule-id`. The row offers the rules of the model as suggestions. |
| Key parts | A list with one row for each name that identifies one record of the rule. The order of the rows is the order of the key. The row offers the names that the `emitRecord` key of the program uses. |
| Data fields | A list with one row for each field of the record. Each row has a name and a type. The row offers `text` and `number` as types, accepts any other type, and saves `text` when the type is empty. |
| Program | The program as a list of steps, with a **Show code** button and an **Edit query** button that opens the query builder in a dialog, for a `generator` or a `transform` rule |

Select **Add dependency**, **Add key part** or **Add data field** to add a row to a list. Select **×** at the end of a row to remove it. To move a row, drag the handle at its start. To move a row with the keyboard, focus the handle and press Space. Press an arrow key to move the row. Press Space to drop it.

The drawer leaves an empty row out when it saves. The drawer refuses a name that appears twice in a list, and shows which name. **Test rule** and **Save** stay disabled until you change it.

A help icon beside each field opens a short explanation on hover and when you tab to it. A message under a field names the problem with that field. A line above the buttons says why **Save** and **Test rule** are off.

A line above the program reads **(no query)**, **1 statement** or the number of statements. The drawer shows each statement of the program as a numbered row, including statements that use `emitRecord` and `groupBy`. A badge on each row says what the statement does, and a generator's last step has the **Emit** badge. A comment shows as a caption, and a disabled statement stays in its place with an **off** badge. **Show code** shows the program as text, with a **Copy** button. Select **Edit query** to edit a statement in the query builder. **Apply** returns the edited program to the drawer, and **Test rule** and **Save** use it. **Cancel** keeps the program as it was. Escape closes only the dialog.

A manual rule has no program. Switch the mode to `manual` and save, and the saved rule loses its program. A `generator` rule needs a program, and the drawer refuses to save one without it.

The last statement of a program must end in the call that its mode needs. A `generator` rule ends in `emitRecord`, and a `transform` rule ends in `save(result)`. When the last statement ends in another call, the drawer shows a message under the program and disables **Save**. The message names the call that the program ends in. It offers two fixes: change the mode, or end the program with the call that the mode needs.

`emitRecord` takes the key first. The data, links and props arguments are optional, so `emitRecord("{ zoneId: item.id }")` creates a record with no data and no links. The fourth argument sets `hide` on the records of that statement, for example `"{ hide: true }"`. It accepts no other key.

Switch a manual rule to `generator` or `transform`, and the program reads **(no query)**. **Edit query** then opens the builder on one starter statement for the mode. A `generator` rule over the model starts from a `where` step that ends in `emitRecord`. A `generator` rule over records starts from a `groupBy` rollup. A `transform` rule starts from a `where` step that ends in `save(result)`. The drawer writes nothing until you select **Apply**, so **Cancel** leaves the rule without a program. A rule that already has a program opens on that program.

A `generator` rule can start from a built-in template. **Edit query** opens on the built-in templates while the program is still the starter, and **Insert** replaces the starter with the template you choose. See [Rule templates](/docs/sightline/usage/shared/rule-templates).

## Make a rule a diagnostic rule

A diagnostic rule is a `generator` rule with **Hide from consumer lists** switched on. Its records are the diagnostics that the Problems view shows. See [Write a diagnostic rule](/docs/sightline/usage/tetra/rule-packages#write-a-diagnostic-rule).

1. Choose `generator` in **Mode**, and select **Hide from consumer lists**.
2. Select **Edit query**. End each statement with `emitRecord`. Give the entity as the key, a `message`, a `severity` and an `entityKind` as data, and the entity as the link.
3. Select **Test rule**. The panel lists the records the rule would create, and the data of each record shows its message.
4. Select **Save**.

## Make a rule a transform

Choose `transform` in **Mode** for a rule that selects rows or computes one value and creates no record. A transform ends in `save(result)`. The drawer hides **Hide from consumer lists**, **Input**, **Depends on**, **Key parts** and **Data fields**, because a transform has none of them.

A run does not execute a transform. **Test rule** runs it over the model. It reports the number of rows that `save(result)` took, or the value when it took one that is not a set of rows. The test writes no record and no journal entry.

Two program features serve a transform. The `set(field, expression)` step writes a field on each row and keeps the `id` of the row. A step such as `count()` produces a value, and `save(result)` keeps that value as the result.

## Roll up records

A rollup is a rule with the input `records`. It reads the records of the rules in **Depends on** and creates records of its own. For example, a rollup can create one record for each zone that more than three relations enter.

In a rollup, the `model` source of the program yields one row for each record of the rules in **Depends on**. A rule never reads its own records. Each row has these fields:

| Field | Content |
|---|---|
| `id` | The identifier of the record |
| `ruleId`, `packageId` | The rule and the package that created the record |
| `key`, `data` | The key parts and the data of the record, as objects. Write `item.key.zoneId` to read a key part. |
| `status`, `severity` | The fields a person owns |
| `closed` | `true` when the rule no longer emits the record, otherwise `false` |
| `state` | `current`, `data-changed`, `source-changed` or `both-changed` |

A rollup that ignores closed records tests `!item.closed`. A rollup that tests `item.status` changes its result when a person resolves a record it reads.

A run executes a rule after the rules it depends on, so a rollup sees the records that its dependencies create in the same run. **Test rule** does the same, using the unsaved result of the dependencies. A rule that depends on a rule that cannot run does not run, and its package is skipped.

Two rules cannot depend on each other, directly or through other rules. **Save** refuses a rule that closes such a cycle and shows the rules in the cycle.

## Test a rule

Select **Test rule** to run the edited rule against the model without saving it. The run writes nothing. The panel under the drawer shows what a real run would do:

- Each record the run would create, change, close or return, with its key
- The number of existing records that change identity
- The diagnostics of the rule

A record is identified by the key parts of its rule. When an edit changes the key parts, the existing records of the rule no longer match. They are closed, and the run creates new ones. The panel then reads, for example, **2 existing records change identity**.

When the program has an error, the panel shows the diagnostic and lists no records. **Save** stays disabled until a test runs without an error, and the line above the buttons says so. Any edit clears the last result, so the failed test no longer blocks **Save**.

## Save a rule

Select **Save** to write the rule to its file in the package. Only that rule changes. The other rules in the file keep their text and comments, and the drawer closes.

When the package changed on disk after the panel read it, the panel says so and reloads the package. The drawer keeps your edit. Select **Save** again to replace the current rule with it.

## The Records tab

The tab lists the records of the model. Select a record to see its journal and the number of Lamina causes that link it.

| Column | Content |
|---|---|
| title | The title of the record |
| rule | The rule that created the record |
| package | The package of that rule |
| state | Current, Changed or Closed |
| status | The status of the record |
| data | The data the rule computed |

The **State** column filter, or a condition on the state field in **Filter**, limits the list to one state. **Columns**, **Filter** and **Group** work as on the Rules tab. A record is Changed when its data or a link differs from the last accepted state. A record is Closed when its rule no longer emits its key.

### Run the rules

Select **Run rules** to see what a run would do. The panel shows the number of new and closed records, for example **3 new, 1 closed**. Select **Confirm** to run the rules and write the records. Select **Cancel** to write nothing.

A run never deletes a record and never changes the fields a person owns. A run does refresh the links of manual records. When the version of a linked entity moved, the record shows Changed, and its data stays as the person set it.

### Accepting a record

The Records tab changes no record. A person accepts a record in the app that uses it. For a finding, that app is Metron's **Findings** tab; for a cause, it is the link of a Lamina cause. A record that a person accepts as a record, such as a Metron finding, returns to Current. A record that only a cause links stays Changed here, because the cause keeps its own accepted state. See [Record links](/docs/sightline/usage/lamina/finding-links).
