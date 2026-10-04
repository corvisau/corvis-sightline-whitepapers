Bulk edit applies one field change to many selected rows in a single action.

Tetra and Lamina offer bulk edit in the Tabular Editor, over fields that come from each entity's schema. Metron offers it in the assessment worklist, over the assessment fields of a requirement and entity pair.

---

## Using it

1. Select two or more rows.
2. Right-click any selected row. The context menu shows only **Bulk edit (N rows)**, where N is the number of selected rows. It replaces the row's normal menu.
3. In the dialog, tick each field to change. Set a value for each ticked field.
4. Click **Apply to N**.

Use any of these ways to build the selection:

- Check the row checkbox to add one row at a time.
- Click a row to select only that row.
- Shift-click a second row to select the range between the two rows.
- Ctrl-click or Cmd-click a row to add it to the selection or remove it.

Right-clicking a row outside the current selection collapses the selection to that row. The context menu then shows the row's normal menu.

## Which fields are offered

In Tetra and Lamina, the dialog lists every field that meets both conditions:

- The field has a basic type: string, number, boolean or enum. These are the same fields that a single grid cell can edit inline.
- Every selected row's type declares the field.

A selection of mixed types therefore narrows the list to the fields that all of those types declare.

An enum field renders as a dropdown of the options that its schema declares. Editing a single cell uses the same dropdown. Either route rejects an invalid value with a visible error. A number field given non-numeric text is one example. A rejected value never enters the pool of unsaved changes.

Metron does not derive its fields from a schema. See the Metron section below.

## Apply semantics

Bulk edit stages its changes in the same pool of unsaved changes that a single inline cell edit uses. It does not write to disk when you apply it. In the Tabular Editor, after you apply an edit:

- Every selected row's cell shows the new value with the usual highlight for an unsaved cell.
- The save bar at the bottom of the surface shows the updated **N unsaved changes** count.
- **Save all** writes the staged edits. You can also save one row at a time, as you do after a normal inline edit.

Metron stages and saves differently. See the Metron section below.

## Tetra

Select rows of the same type, or of mixed types. A device and a channel selected together is one example.

The context menu that **Bulk edit (N rows)** replaces holds **Edit**, **View Code**, **Delete** and the add-child items.

A selection that mixes types offers only the fields that all selected types declare. For example, a device and a channel together offer only the fields that both schemas declare. `description` is common to all eight entity schemas. A channel-only field such as `protocol` disappears once a device row joins the selection.

## Lamina

Select rows of the same type (cause, event, outcome or control), or of mixed types.

The context menu that **Bulk edit (N rows)** replaces holds **Edit**, **View Code** and **Delete**.

A selection that mixes types offers only the fields that all selected types declare. A cause and a control together offer only the shared fields, such as `label`. A control-only field such as `controlType` or `category` appears only when every selected row is a control.

The enum fields include `category`, `controlType` and `currentEffRating`. Each renders as a dropdown of the schema's declared options.

## Metron

Metron bulk edit works on pairs in the assessment worklist. A pair is one requirement on one entity. Bulk edit changes assessment fields on many pairs at once.

Bulk edit copies values once. To keep many pairs on one assessment, link them to a [shared assessment](/docs/sightline/usage/metron/shared-assessments). A later edit then reaches all of them. Bulk edit can link the pairs.

Every edit in the worklist stages into a shared pool of unsaved changes instead of writing immediately. This covers inline cells, bulk edit and the Assess drawer. Nothing reaches the disk until you click **Save all**, or **Save** in the Assess drawer. See the Saving your changes section below.

### Using it

1. Select two or more pairs in the worklist. Check the row checkbox, which is the same checkbox that selects pairs for **Promote to finding**. The click, Shift-click and Ctrl-click or Cmd-click selection methods work as described earlier.
2. Right-click any selected row. The context menu shows only **Bulk edit (N pairs)**, in place of the usual **Assess** item.
3. In the dialog, tick each field to change and set a value for each.
4. Click **Apply to N**. This stages the change on every selected pair. The dialog closes at once, and nothing is written to disk.

Right-clicking a row outside the current selection collapses the selection to that row. The menu then shows the normal single-pair **Assess** item.

### Inline cell editing

To edit a single pair, click its **Outcome**, **State** or **Notes** cell in the worklist grid. You do not need to select the row or open the dialog.

- **Outcome** and **State** render as dropdowns. Each dropdown starts with the pair's current or already-staged value. Choosing a different option stages it, and the cell shows the highlight for an unsaved cell.
- **Notes** becomes a multiline text box. Edit the text, then click away or press Tab to stage it.
- Press **Escape**, or click away without choosing an option, to cancel. Nothing stages.

Inline editing covers the same three fields as bulk edit, described in the Fields section below. It applies them to one pair through the same write path. The gates that bulk edit applies work the same way, and they run at Save time, when the edit is written:

- Moving a pair to a done status requires an outcome.
- Editing the outcome or the comment also requires an existing outcome.

The cells of a pair in a done status cannot be edited inline. Reopen the pair in the Assess drawer first. Bulk edit and the drawer require the same status-picker confirmation to leave a done status.

The **Req** and **Entity** columns are never editable, inline or in the dialog. They form the pair's composite key: which requirement, on which entity.

### Saving your changes

Every staged edit lands in one pool that is keyed by pair. The edit can come from an inline grid cell, the bulk edit dialog or the Assess drawer. A field edited in the drawer and another field edited inline on the same pair count as one unsaved change to that row.

- **Unsaved-change bar.** A bar appears above the worklist whenever anything is staged. It shows the count of unsaved changes, with the **Save all** and **Discard all** buttons.
  - **Save all** writes every staged row. When everything has settled, it writes the assessment binding file once, however many pairs or fields you staged.
  - **Discard all** clears every staged edit and writes nothing. The cells and the drawer revert to their last-saved values.
- **The Assess drawer.** The drawer has its own **Save** and **Discard** footer. The footer saves or clears only the pair that is open in the drawer. It does not touch other staged rows.
- **Partial success.** If **Save all** includes a row that fails a gate, the other rows still save. An example is a done transition with no outcome. The failing row stays in the unsaved-change count. The reason shows near the worklist, so you can fix the row and save again.
- **Editing without a status change.** Outcome, evidence and notes edits stay staged and savable on their own. An edit to those fields on a `draft` or `in-progress` pair is not lost.

### Fields

| Field | Stored as | Notes |
|---|---|---|
| Assessment state | `state` | The options come from the `statuses` list of the package's assessment scheme. A package that does not customise them gets the three built-in fundamentals. |
| Assessment outcome | `outcome` | The options come from the `outcomes` list of the assessment scheme. |
| Outcome comment | `notes` | Free text. |
| Shared assessment | `sharedRef` | Offered when the assessment has at least one [shared assessment](/docs/sightline/usage/metron/shared-assessments). Every record is offered, whatever the requirement. **None** unlinks the pair. |

State, outcome and comment are always available. Metron's assessment state and outcome are package-defined values that the assessment scheme sets. They are not fixed schema enums, so they do not depend on which pairs you select.

### Apply semantics

- **Staging never fails.** **Apply to N** always stages every selected pair and closes the dialog. Nothing is written yet, so nothing can be rejected at this point.
- **A done status requires an outcome, checked at Save time.** Suppose you tick only **Assessment state** and target a done status. If a pair has no outcome, neither existing nor staged in the same bulk edit, that pair fails to save. It stays in the unsaved-change count with the reason shown near the worklist. The other staged pairs still save.
- **Untouched fields stay as they are.** Ticking only **Outcome comment** does not change a pair's existing outcome or evidence.
- **Ticking Shared assessment changes only the link.** The pair's own outcome, comment and evidence stay as they were. The pair shows the shared values wherever its own values are empty. A shared assessment that supplies an outcome counts as an outcome for the pair. Ticking **State** for a done status in the same edit then succeeds.
- **Ticking outcome or comment also requires an existing outcome.** This holds for a pair that has never been assessed, and not only for a done transition. The write path always sends an outcome value. A pair with no outcome, existing or newly staged, fails to save in the same way as a done transition with no outcome.
- **The dialog always closes and the selection always clears on Apply.** The Saving your changes section describes what happens next.

### Interaction with Promote to finding

Bulk edit and the promote-to-finding flow share the worklist checkbox column. Both start from the same right-click menu.

- Right-click a single eligible pair, checked or not, to see **Promote to finding** beside the usual **Edit...** item. A one-off promote needs no checkbox.
- Check two or more eligible rows, then right-click any of them. The menu shows **Promote N to finding** beside **Bulk edit (N pairs)**. N counts the eligible subset of the checked rows. A checked ineligible row, such as one still in `draft`, does not change that count.
- An ineligible row never shows a **Promote** item, checked or not.

See [Findings](/docs/sightline/usage/metron/findings) for the promote flow.
