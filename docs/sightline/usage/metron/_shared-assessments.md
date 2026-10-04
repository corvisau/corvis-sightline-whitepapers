A shared assessment holds an outcome, a comment, inline evidence and linked
evidence. Many (requirement × entity) pairs link to one shared assessment, and an
edit to it reaches every linked pair. Use one when the same control is assessed
identically across many zones or systems.

A shared assessment applies to any requirement. Two pairs under different
requirements can link to the same one.

## How a pair resolves

A linked pair shows the shared assessment underneath its own values.

| The pair's field | The pair shows |
|---|---|
| Empty | The shared assessment's value |
| Set | The pair's own value |

An empty field is a blank comment, no outcome, no evidence or no linked evidence.
Clear a field on the pair to return it to the shared value.

## What a shared assessment carries

| Field | Behaviour on a linked pair |
|---|---|
| Outcome | The pair's own outcome wins when set. |
| Comment | The pair's own text wins when it is not empty. |
| Evidence | A non-empty list on the pair replaces the shared list. The two lists never combine. |
| Linked evidence | Follows the same replacement rule as evidence. |

The assessment state stays on the pair. Draft, in progress and done are
per-pair workflow, so a shared assessment never sets them.

## Author a shared assessment

The **Shared** tab sits beside **Evidence**.

1. Select **+ Add shared assessment**.
2. Select **Edit...** on the new row.
3. Enter a label. Set the outcome, comment, evidence and linked evidence.
4. Select **Save all**.

The grid edits **Label**, **Outcome** and **Comment** inline. **Edit...** opens a
drawer for the evidence fields, which a grid cell cannot hold. A row needs a
label before it can save.

Two columns are read-only:

| Column | Meaning |
|---|---|
| Version | Metron raises it when the outcome, comment, evidence or linked evidence changes. A rename does not raise it. |
| Used by | The number of pairs that link the shared assessment, across every requirement. |

Filter, group, search and column visibility work as on the **Evidence** tab. To
delete several rows, select them and right-click.

## Link a pair

Link a pair in the Assess drawer:

1. Open the pair with right-click, then **Edit...**.
2. Choose a record in **Shared assessment**.
3. Select **Save**.

The picker appears once the assessment has at least one shared assessment, and it
lists all of them. Choose **None** to unlink the pair. A pair that is not done then
shows only its own values, so an outcome or comment it inherited disappears from
the row. A done pair keeps what it showed, as its own values.

The drawer previews the shared content as soon as you choose. A **from shared**
marker sits above each field that the record supplies. A done pair is read-only.
Reopen it first.

To link many pairs, select them and choose **Bulk edit**. Tick **Shared
assessment**. Pick a record or **None**. Select **Apply**. The change stages
like any other edit and saves with **Save all**.

The **Shared** worklist column shows which record each pair links. It is off by
default. Turn it on with **Edit columns**.

## Override and revert

Type in a field on a linked pair to override it. The pair keeps its own value, and
its **from shared** marker disappears. A later edit to the shared assessment leaves
that field alone.

Clear the field to revert it. The pair shows the shared value again, and the marker
returns.

A linked pair can move to done with an inherited outcome. A pair with no outcome of
its own or from its record cannot.

## Completed pairs

When a pair enters a done status, Metron stamps it with the shared assessment's version. A later
change to the shared content raises the version above that stamp, and the pair shows
the **Reval** warning.

The pair follows the shared content at once, so its outcome in the dashboard, the
exports and the reports moves with the record. The warning marks the pairs whose
finished assessment changed. Right-click a flagged pair and choose **Revalidate**
to accept the change. Metron stamps the pair with the current version when you revalidate it.

Two link changes on a done pair raise the same warning:

- Linking a done pair to a shared assessment
- Switching it to a different one

A done pair that you unlink keeps the values it showed as its own, so it raises no warning.

## Delete a shared assessment

When you delete a shared assessment, Metron unlinks every pair that used it.

| Pair | Result |
|---|---|
| Done | Keeps the outcome, comment and evidence it showed, now as its own values. |
| Not done | Falls back to its own values, which are usually empty. |

When you delete a general-evidence record, Metron removes its id from every shared
assessment that cited it.

## Reports and exports

The dashboard, the CSV export, the JSON export and the report envelope read the
resolved values. A linked pair appears with its shared outcome, comment and linked
evidence, as if the pair held them. No report field or contract version changes.

## File format

Shared assessments live in the `*.assessment.yaml` file, beside `generalEvidence`.

```yaml
sharedAssessments:
  - id: sa-1
    label: Zone baseline
    version: 2
    outcome: mil-3
    notes: Standard zone control
    evidenceRefs: [ev-1]
pairs:
  - requirementId: IAM-1
    entityId: d-1
    state: done
    outcome: null
    evidence: []
    sharedRef: sa-1
    assessedSharedVersion: 2
```

A pair writes only its own values. The linked pair above holds `outcome: null`, and
the outcome it shows comes from `sa-1`.

`sharedAssessments` and `sharedRef` are optional, and a file without them loads
unchanged. No migration is needed.

A `sharedRef` that names no record raises a warning in the Problems panel. The pair
shows only its own values, and Metron does not flag it for revalidation.

## Choosing between reuse tools

| Tool | Reuse | Carries |
|---|---|---|
| Shared assessment | Live link to one record | Outcome, comment, evidence, linked evidence |
| [Bulk edit](/docs/sightline/usage/shared/bulk-edit) | One-time copy onto the selection | State, outcome, comment |
| [General evidence](/docs/sightline/usage/metron/general-evidence) | Reference records cited by id | Evidence records |
| [Attribute sets](/docs/sightline/usage/metron/attributes) | Target attributes shared across requirements | Attributes, not outcomes |
