The **Findings** tab lists the findings of the assessment's Tetra model. A finding
groups assessed pairs under one title, one status and one severity. Each pair a
finding covers is a member of it. Create a finding from flagged worklist outcomes
(see the worklist's **Promote to finding** button on
[the worklist filtering page](/docs/sightline/usage/metron/worklist-filtering)). Every field is directly
editable in the grid, and the tab shares the same toolbar mechanism as the
Assessment Worklist tab.

The [Records](/docs/sightline/usage/metron/rule-findings) panel lists the same findings and every
other record of the model, with their state and journal. The
Findings tab has one row for each finding, whatever the number of its members.

## Fields

| Column | Editable as | Notes |
|---|---|---|
| Label | text | Short finding title. |
| Description | multi-line text | Free-form detail. |
| Status | select (open / reviewed / resolved) | The human review lifecycle. |
| Severity | select (low / medium / high / critical) | Author-assigned; defaults to `medium`. |
| Tags | chip add/remove | Free-text labels; type a tag and press Enter to add, click a chip's `×` to remove. |
| Members | read-only list | The assessed pairs the finding covers, each with the state of its link and a **Remove** button. |
| Stale | read-only badge | Shows `stale` when the link of any member has drifted. Never editable. |

Click any editable cell to enter edit mode. Text fields stage on blur or
Enter, selects stage on choosing an option, and tags stage on every
add/remove. A staged cell shows a dirty highlight. Nothing writes to
disk until you click **Save all** in the shared save bar at the bottom of the
assessment. That bar appears whenever any Worklist, Findings, or Evidence edit
is unsaved. **Discard all** drops every staged edit across all three tabs.

## Members

A member is the assessment record of one requirement and one entity. The Members
column lists each member as its requirement id and entity id, followed by the state
of its link:

| State | Meaning |
|---|---|
| `new` | The link has no accepted pin. |
| `current` | The member has not changed since the link was pinned. Adding a member pins it. |
| `data-changed` | The outcome or the attribute states of the pair changed. |
| `source-changed` | The entity or the requirement has a newer version. |
| `both-changed` | The data and the source both changed. |
| `closed` | A rule closed the member's record. |
| `missing` | The record no longer exists. |
| `unreadable` | The register of the member's assessment cannot be read. |

The Stale badge shows for every state except `new` and `current`. A member from
another assessment of the same model appears in the same list.

**Remove** on a member removes its link. The member's own record stays as it was.

## Create a finding and add members

**Promote to finding** on the worklist opens a dialog with two choices:

- **Create finding** adds one finding. Its members are the selected pairs, and the
  label, description, severity and tags are the ones you enter.
- **Add to existing** adds the selected pairs as members of the finding you choose.
  A pair that is already a member is not added twice.

A finding is saved with the assessment's Tetra model. The dialog has no
location to choose.

A member is a whole pair. The attributes that the assessor recorded on the pair are
not split out, so a pair with two unmet attributes gives one member. The
[Coverage](/docs/sightline/usage/metron/coverage) tab shows that member as a whole-practice row.

An earlier version stored a finding for one attribute in a different form. This
tab does not list that finding, and nothing migrates it. Create a finding over
the same pairs to replace it.

## Toolbar (Filter / Group / Saved filters / Columns)

The Findings tab's toolbar uses the same popover
components as the worklist tab. The **`[Filter]`**, **`[Group]`**, **`[Saved
filters]`** and **`[Edit columns]`** controls operate on findings instead of
pairs. Each has its own independent filter program, group-by field, saved-filter
list and column-visibility state. The "Filter + Group" section of
[Worklist Filtering, Grouping, and Slices](/docs/sightline/usage/metron/worklist-filtering) covers the
Basic and Advanced tabs and how conditions compile to a Bonsai program.

### Available Basic-tab filter fields

| Field | Type | Notes |
|---|---|---|
| Status | enum (open / reviewed / resolved) | |
| Severity | enum (low / medium / high / critical) | |
| Stale | boolean | Matches the Stale column's badge. |

`Tags` is not offered as a Basic-tab field, because it is a list of free-text
strings and not a single scalar value. The worklist excludes `attributes` for
the same reason. The Advanced tab still reaches it
(`item.tags |> hasAny('...')`).

### Available Group-by fields

The Group menu offers **Status** and **Severity**, both low-cardinality. It does
not offer Label, Description or Tags, which are too high-cardinality or not a
scalar column value to be useful to group by.

## Bulk review actions

Check the row checkboxes of two or more findings to show a bulk action bar in
the toolbar. The buttons **Mark N reviewed** and **Resolve N** stage the same
status transition on every selected finding at once. This equals editing the
Status column inline on each. Nothing writes to disk until you click
**Save all**. The selection clears immediately after the action, and whenever a
fresh set of findings loads (a reload, or switching assessments).

## Jumping in from Tetra

Select **Jump to Metron** in a Tetra diagram node's findings popup. Metron
opens the assessment on this Findings tab with the selected finding checked in
its row checkbox. A jump into a new tab applies no filter, so every finding
for the assessment is visible.

- **The selection is a starting point, not a lock.** Uncheck it, or check
  others, like any other bulk selection. Nothing re-applies it, and it clears
  when the findings data reloads.
- **One tab serves every entity.** A second jump for the same assessment
  reuses the open tab. The tab switches to Findings, clears any Findings
  filter and grouping, checks the new finding and scrolls to its row. The tab
  title stays the model name, because the tab no longer belongs to one entity.

See [Findings overlay](/docs/sightline/usage/tetra/findings-overlay) for the Tetra-side trigger.

## Accept

**Accept** on a stale finding opens a confirmation. Confirming pins every member at
its current state. Each link records the live entity version, the live requirement
version and the current outcome of the pair. The journal records the acceptance. The
status and the other person-owned fields stay as they were. A member whose record no
longer exists keeps its `missing` state and has no pin to move.

## Saving

Every field above stages into one pool of unsaved changes shared with the Worklist and
Evidence tabs. **Save all** flushes every staged finding, together with any other
staged Worklist or Evidence edits. The save applies the edits in order and stops at the first refused write.
Another writer that changed the same field first answers a conflict and leaves
the rest staged. Edits to different fields of one finding, or to different findings,
do not conflict.
**Discard all** drops every staged edit across all three tabs and restores the
last-saved values.
