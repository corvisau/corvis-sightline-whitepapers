The **general evidence** table holds assessment-wide reference records (a policy
document, a meeting note, a standard). A record is authored once and cited from
any number of pairs, so the same evidence is not re-entered on every outcome
record.

General evidence is separate from, and compatible with, the inline per-pair
evidence on the Assess drawer (`kind`/`value`/`label`). Use inline evidence for
something specific to one pair. Use general evidence for something shared across
the assessment.

---

## The Evidence tab

The assessment shell's **Evidence** tab (after Worklist / Dashboard /
Findings, always visible) hosts the general-evidence Tabular Editor, a
grid that follows the Worklist tab's editing model:

| Column | Meaning |
|---|---|
| Title | Required, non-empty. Primary display - doc title, meeting-note name. |
| Kind | `file` (a filename/path), `url` (a link), or `note` (a free-text reference). |
| Value | The locator: a URL or filename. A bare `note` may leave this blank. |
| Version | Document version label, when applicable. |
| Retrieved | Retrieved/accessed date, when applicable. |
| Comment | Free-text comment. |
| Open | A hyperlink for `file`/`url` rows with a value - see below. Blank for `note` rows. |

Click any cell to edit it inline, as on the Worklist tab. The edit stages on
blur or Enter, and the shared save bar shows an unsaved-changes indicator. The
save bar appears whenever any Worklist, Findings, or Evidence edit is unsaved.
Nothing writes to disk until you click **Save all**.

Click **+ Add reference** to create a row with a new stable id. Click the **⌫**
button on a row to remove it. Editing an existing row keeps that row's id; only
adding a row creates a new one.

A new row with a blank title stages nothing, because a title is required. The
row still displays locally until you give it a title. Adding a blank row does not
discard another staged edit. The table stages as one atomic array, which leaves
the incomplete row out until it has a title.

Use the **Columns** picker to show or hide the Title, Kind, Value, Version,
Retrieved and Comment columns. The Open and Remove columns always show. Use the
grid's search box to filter rows by the text of any visible column.

### Filtering, grouping, and bulk actions

The toolbar also offers the same query-builder controls as the Worklist tab:

- **Filter**: narrow the visible rows with one or more Kind, Title, Value,
  Version, Retrieved or Comment conditions (Basic tab). Alternatively,
  hand-author a Bonsai program (Advanced tab). The filter composes with the free-text search box
  above.
- **Group**: group rows by **Kind** or **Version**. Choose **None** to ungroup.
- **Saved filters**: save the current filter as a named shortcut, or apply or
  remove a previously saved one. Saved filters are personal, stored locally and
  independent of the Worklist tab's saved filters.

Each row also has a selection checkbox. Select two or more rows, then
right-click any selected row to open:

- **Bulk edit (N)**: opens a dialog to set Kind, Version, Retrieved or Comment on
  every selected row at once. Title and Value are edited per row only, because a
  title names one row and a value is a per-row locator. Only the fields
  you tick change.
- **Delete N selected**: stages the removal of every selected row in one action.
  The linked-pair cleanup matches removing a single row (see the section Deleting
  a linked record) and applies once the staged edit is saved.

Selection is independent of the active filter. Rows selected before filtering
stay selected, and stay in a bulk action, even when the filter hides them.
Editing or deleting a single row uses the inline cell edit and the **⌫** button.
The right-click menu appears only for two or more selected rows.

## Saving

Every edit above (cell edits, add, remove, bulk edit and bulk delete) stages the
whole table as one entry. The entry goes into the shared pool of unsaved changes that the
Worklist and Findings tabs also use. **Save all** persists the table in a single atomic write, together
with any other staged Worklist or Findings edits. **Discard all** drops every
staged edit across all three tabs and restores the last-saved table.

### Opening the evidence file

The **Open** column renders a link for any row whose `value` is set:

- `file`: opens the referenced file with the OS's default application
  (resolved relative to the assessment `*.assessment.yaml`'s own directory,
  or as an absolute path).
- `url`: opens the link in the default browser (a real `target="_blank"`
  anchor).
- `note`: no link.

The Open column becomes plain text when the panel is read-only.

## Linking a record to a pair

Open a pair's Assess drawer. Below the inline **Evidence** editor, the **Linked
evidence** section lists each record already linked to the pair as a removable
row. An autocomplete box below the list adds more records. Type to filter the
assessment's general-evidence records by title. Pick one to link it. Adding or
removing a row persists through the same apply path as the pair's outcome,
evidence and notes. There is no separate save button. The section is omitted when
the assessment has no general-evidence records.

Inline evidence and linked general evidence are independent: a pair can carry
either, both or neither. A pair linked to a
[shared assessment](/docs/sightline/usage/metron/shared-assessments) shows the linked evidence that the
shared assessment cites when the pair has none of its own.

## Deleting a linked record

Deleting a record in the Evidence tab removes it from the table and clears its id
from every pair that linked it. No dangling reference remains in a pair's Assess
drawer, the report or an export. It also clears the id from every
[shared assessment](/docs/sightline/usage/metron/shared-assessments) that cited it.

## Report and exports

- **Report envelope** (contract v5): `assessment.generalEvidence[]` carries
  every authored record. A linked pair's block carries only its ids in
  `evidenceRefs`, and an unlinked pair omits the key.
- **CSV export**: an `evidence_refs` column lists each pair's linked ids,
  joined with `;`. CSV is per pair, so it carries only the link ids. The full
  records are in the JSON and report exports.
- **JSON export**: a top-level `generalEvidence` array beside the
  `rollup` tree carries every authored record in full.

## Data model

```yaml
# *.assessment.yaml
generalEvidence:
  - id: ev-1
    kind: note
    title: Kickoff meeting notes
    retrievedAt: "2026-07-15"
    comment: Scope agreed with security lead
pairs:
  - requirementId: IAM-1
    entityId: d-1
    # ...
    evidenceRefs: [ev-1] # optional; absent for an unlinked pair
```

`generalEvidence` defaults to `[]` and `pairs[].evidenceRefs` is optional, so
every existing `*.assessment.yaml` loads unchanged without migration.
