Sightline Lamina, Tetra and Metron enforce their commercial licence terms in the app. A missing, invalid, expired or feature-lacking licence blocks saving. Viewing and export stay usable without a licence.

A commercial licence names the apps it unlocks. Editing is licensed per app. One licence covers every Sightline app on a machine.

## What a licence contains

A licence names the person or organisation that it is issued to, and it carries an issue date and an expiry date. A licence that lacks an expiry date, or that names no app, is invalid.

## Per-app grants

The licence must name the app that you want to edit in.

| Licence names | Unlocks editing in |
|---|---|
| Lamina | Lamina |
| Tetra | Tetra |
| Metron | Metron |

A licence can name any subset of the apps. A licence for one app leaves the other apps able to view and report, but not to save.

When a licence is valid but does not name an app, the licence badge in that app reads `Licensed to: <name> (no editing in <app>)`. The badge shows this state before you try to save. The exact badge text for each app is in the app sections below.

## Metron data in Tetra

Tetra reads Metron findings and coverage to draw the findings badges and the coverage overlay. This read needs a licence that carries the `view:metron` grant. A licence that lacks it leaves Tetra fully usable. The findings badges do not load, and the coverage overlay shows a notice that gives the reason.

The grant covers the read only. It does not unlock editing in any app.

## Editing without a licence

Every editing control stays enabled without a licence, with the exceptions listed for each app below. You can make changes, but the app does not save them.

The first pending change shows a banner: `Editing is not licensed. You can make changes, but they will not be saved.` The banner goes when no change is pending.

**Save all** opens the **Save changes** dialog. The dialog lists each pending change by the label or id of the entity, then what changed. In the dialog:

- **Save** writes the changes. Without a licence, **Save** is disabled and the dialog shows `Editing requires a valid commercial licence.`
- **Cancel** closes the dialog and keeps every change pending.

## Where the licence is checked

The app checks the licence again when a change is written, whatever the panel shows. The check covers every write to a model file, a data slice or a query template. Changes that wait in the pending pool commit together, so one **Save** is one licence check.

## When a save is refused

A refused save reports its reason where you asked for it, and the refused change stays pending. When the app refuses a save in the **Save changes** dialog, the dialog stays open and shows the reason. Fix the cause, then save again.

A write that fails on a disk error or a validation refusal reports itself the same way. The pool save bar shows the reason beside its own button.

## Managing your licence

Select the licence badge in the bottom-right corner to paste, view or clear the licence string. The apps on a machine share one licence, so pasting it in one app puts it in force in the others.

Pasting or clearing a licence takes effect without reopening the panel. In the app where you paste it, the banner and the **Save** button in the dialog follow the new licence straight away. The app sections below list any further controls that follow it.

## A read-only view is a different thing

The licence answers whether a change can be saved. It never stops you making a change.

A second, independent setting answers whether a surface can be changed at all. With that setting off, every control that would edit the model is disabled.
The two settings never read each other. An unlicensed app is fully editable and refuses at **Save**. Controls that only read an entity's properties or source stay available in both, because reading is not editing.

Every panel opens editable.

## Revocation

A licence can be revoked. A revoked licence blocks editing for the rest of that VS Code session. The activation ping that runs beside this check is described in [Telemetry](/docs/sightline/usage/shared/telemetry).

## Leaving with changes pending

Tetra and Lamina keep pending changes when you leave, and ask first.

**Closing the panel.** A model panel with pending changes is marked dirty, so VS Code shows its own unsaved-changes prompt when you close the tab. Choosing **Save** there runs the same commit as the **Save changes** dialog, licence check included. If the licence check refuses the save, the panel stays open and your changes stay pending.

This covers a model that you open the normal way, by selecting it. A panel that you open from the command palette has no document behind it, so VS Code shows no prompt for it. Save before you close a panel opened from the command palette.

**A reload from the host.** A refresh that would rebuild the diagram asks before it discards pending changes.

**Switching inside the panel.** Switching applies at once, and your pending changes stay staged and visible after the switch. What you can switch differs by app, as the Tetra and Lamina sections below list.

## Undo, redo and the journal

Tetra and Lamina share these controls.

**Undo** and **Redo** sit beside the save controls. They step back and forward through the changes you staged this session, one at a time. When you undo the only staged edit, the controls stay on screen so that Redo remains reachable, although nothing is pending.

Staging a new change after an undo drops whatever Redo would have restored.

**Journal** is a section of the Sightline sidebar, below Model Tree. It starts collapsed and appears once you open a model. Expand it to see every staged change in the order you made it, beside the props drawer rather than behind it. An undone entry stays listed, struck through, so you can see what you stepped back past. The journal has its own **Undo** and **Redo** buttons. It follows the editor you are looking at, and its buttons act on that model. The journal clears when you commit.

Undo reaches only what is staged. Recover an earlier commit with git.

## Structural changes wait for Save

Structural changes wait in the pending pool with your field edits, and the unsaved count includes them. A change is visible at once, although nothing has reached disk. A new entity appears on the canvas, in the Tabular Editor and in the drawer as soon as you create it. A deleted entity disappears as soon as you delete it.

In the Tabular Editor, a row with any pending change shows the unsaved highlight across the whole row, and each changed field's cell shows it too when its column is shown. A field edit in a hidden column still marks its row. **Undo** and **Discard all** clear the highlight.

**Save** in the **Save changes** dialog performs every pending change. A refused change stays staged. The app sections below list which actions wait.

## Tetra

**Badge.** A licence that does not name Tetra shows `Licensed to: <name> (no editing in Tetra)`.
**Without a licence.** Viewing and export stay fully usable. Every editing control stays enabled. This includes the node, edge and pane menus, and **New** and **Bulk edit** in the Tabular Editor. The menu entry that opens the properties drawer reads **Edit…**. **Save all** appears in the pool save bar below the diagram and below the Tabular Editor. The **Save changes** dialog lists each change by the label and id of the entity.

Collapsing a zone or group is a pending change like any other. It stages onto the active data slice and waits for **Save**. Without a licence you collapse as freely as you read, and you meet the refusal once, in the **Save changes** dialog. A collapse made with no active data slice persists nothing, licensed or not.

**Refused saves.** The pool save bar shows the reason beside its **Save all** button. The **Save** button of the stacked channel and flow editor drawer keeps the drawer open and keeps your edit in the fields. It shows the reason above its footer.

**Leaving with changes pending.** You can switch the data slice or the surface mode without a prompt. **Undo** and **Redo** sit in the pool save bar, which stays in place when you switch between the diagram and the Tabular Editor.

**Structural changes.** Creating, deleting, grouping, moving, promoting and reordering all wait for **Save**. A pending move highlights the row, its **parent** cell when that column is shown, and the **Parent** field in the properties drawer. A cascade delete records the dependents that you ticked when you confirmed. If you stage later changes that alter what depends on that entity, the recorded list is the one that saves.

**Read-only view.** **Edit…** and **View Code** stay available with the licence off and with the read-only setting on.

**Managing your licence.** In the app where you paste a licence, the banner and the **Save** button in the dialog follow it at once.

## Lamina

**Badge.** A licence that does not name Lamina shows `Licensed to: <name> (no editing in Bow-Tie)`. The badge shows one of these states:

- `(UNLICENSED)`
- `(LICENSE INVALID)`
- `Licensed to: <name> (LICENSE EXPIRED)`
- `Licensed to: <name> (no editing in Bow-Tie)`
- `Licensed to: <name>`, with the licence id, the issue date and the expiry

**Without a licence.** Viewing, findings browsing and report generation stay fully usable. Every editing control stays enabled. **Save all** appears in the pool save bar below the diagram and below the Tabular Editor. The **Save changes** dialog lists each change by the label of its cause, event, outcome, control or annotation. A data slice or query template appears by its own label.

**Leaving with changes pending.** You can switch the data slice or the surface mode without a prompt. Opening a second create surface asks nothing either. Your changes stay pending. **Undo** and **Redo** sit in the pool save bar, which stays in place when you switch between the diagram and the Tabular Editor.

**Structural changes.** These actions wait for **Save**:

- A new cause, event, outcome, control or annotation from the properties drawer
- The Add Risk wizard, which creates a linked cause, event and outcome
- A finding that you promote to a cause
- The deletion of an entity from the canvas, from the context menu
- A change to the style, type, text or placement of an annotation
- A control paste, which always mints a new control id
- A control that you drag to a new position on its cause or outcome

The link that a promoted finding records to its new cause still writes to disk at once. The cause itself waits for **Save**.

A new entity that waits for **Save** shows on the canvas even when the active data slice would exclude it. The slice filter applies again when the model reloads after **Save**.

**Read-only view.** These stay available with the licence off and with the read-only setting on:

- `Edit`
- `View Code`
- `Edit Target`
- `View Code of Target`
- Copy, because the clipboard is not the model

**Managing your licence.** In the app where you paste a licence, the banner and the **Save** button in the dialog follow it at once.

**Revocation.** A revocation applies to the current VS Code session only. A new session checks again.

## Metron

**Badge.** A licence that does not name Metron shows `Licensed to: <name> (no editing in Metron)`.
**Without a licence.** Viewing, the worklist and findings browsing stay fully usable. Every editing control stays enabled, except two commands. These controls work without a licence:

- **+ Add** in the requirement editor, and **+ Add rule**, **+ Add attribute set** and **+ Add scheme** in the package editors
- **Edit** and **Delete** in the row menu of each editor
- The bulk actions on the Findings tab
- The fields of the properties drawer

Without a licence, **+ New assessment** and **Run assessments** are disabled, and their tooltip gives the reason.

The banner shows at the top of the editor. **Save all** appears in the pool save bar only. The **Save changes** dialog lists each change by its id. A pair reads as `<requirement id> × <entity label>`.

**Refused saves.** The pool save bar shows the reason beside **Save all**. A write refused by an assessment business rule reports itself the same way.

**Structural changes.** A create or a delete stages the same way as a field edit. This applies to a requirement, a rule, an attribute set and a scheme. A new entry appears in its grid at once, and a deleted entry leaves the grid at once. Neither reaches disk until you select **Save** in the **Save changes** dialog.

Package meta, the scoring-scheme picker and the `uses:` list also wait for **Save**. They appear in the **Save changes** dialog as `Package meta` and `Packages`, and they commit with the rest of the batch under one licence check. A commit that carries a meta edit and a `uses:` change writes both or neither.

**Read-only view.** The row `Edit` actions in all four package editors stay available with the licence off and with the read-only setting on.

A `done` assessment pair keeps its own lock, separate from the licence and the read-only setting. You reopen the pair before you edit it. The reopen route stays available in a licensed, editable Metron. The route is closed in a read-only Metron, because a viewer has no way back out of `done`.

**Managing your licence.** In the app where you paste a licence, the banner, the **Save** button in the dialog and those two commands follow it at once.
