The **Findings** tab lists the findings of the assessment's Tetra model that name
an entity and a requirement. A finding is created by promoting one or more
flagged worklist outcomes (see the worklist's **Promote to finding** button on
[the worklist filtering page](/docs/sightline/usage/metron/worklist-filtering)). Every field is directly
editable in the grid, and the tab shares the same toolbar mechanism as the
Assessment Worklist tab.

Each finding is a manual record of the rule package `assessment-findings`. The
[Records](/docs/sightline/usage/metron/rule-findings) panel lists the same findings and every other
record of the model, with their state and journal. The Findings tab has one
row for each requirement, entity and attribute, so a promote of a pair with two
unmet attributes adds two rows.

## Fields

| Column | Editable as | Notes |
|---|---|---|
| Label | text | Short finding title. |
| Description | multi-line text | Free-form detail. |
| Status | select (open / reviewed / resolved) | The human review lifecycle. |
| Severity | select (low / medium / high / critical) | Author-assigned; defaults to `medium`. |
| Tags | chip add/remove | Free-text labels; type a tag and press Enter to add, click a chip's `×` to remove. |
| Stale | read-only badge | Shows `stale` when the finding's entity or requirement has a newer version, when the assessed outcome changed, or when a run left the finding **Source changed** or **Data changed**. Never editable. |

Click any editable cell to enter edit mode. Text fields stage on blur or
Enter, selects stage on choosing an option, and tags stage on every
add/remove. A staged cell shows a dirty highlight. Nothing writes to
disk until you click **Save all** in the shared save bar at the bottom of the
assessment, which appears whenever any Worklist, Findings, or Evidence edit is unsaved. **Discard all**
drops every staged edit across all three tabs.

## Where findings are stored

A finding lives in the **register** of the assessment's Tetra model, under the
rule package `assessment-findings`. The promote dialog has no register to choose:
**Create finding** and **Add to existing** both write there. A promote needs the
package. A requirement package that is bound to the model lists
`assessment-findings` in `uses:`.

**Add to existing** creates one finding for each selected pair and attribute, and
each copies the label, description, severity and tags of the finding you chose. A
finding holds one pair, so it cannot take more links.

## What a promoted finding cites

Promoting a pair attaches every attribute the assessor recorded as **Not met**
or **Partial** on it. A pair with two unmet attributes therefore gives two
independently trackable gaps. A pair with nothing recorded, or with everything
**Met**, promotes as a single whole-practice reference. See the section *What
promoting a pair attaches* on the
[Per-attribute assessment](/docs/sightline/usage/metron/attribute-assessment) page.

Each attribute name becomes the `attributeName` part of its finding's key, and a
pair with none becomes one finding with no attribute name.

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

**Accept** on a stale finding opens a confirmation. Confirming runs the model's
Metron rules, which refresh the links of manual records, sets the finding's assessed
outcome to the live outcome and accepts the finding. Its references then pin the
current entity and requirement versions, and the journal records the acceptance. The
status and the other person-owned fields stay as they were. A run covers every Metron
finding of the model, so other findings whose links moved show **Source changed**
afterwards. A finding whose entity no longer exists in the model keeps its stale badge
and offers no Accept.

## Saving

Every field above stages into one pool of unsaved changes shared with the Worklist and
Evidence tabs. **Save all** flushes every staged finding, together with any other
staged Worklist or Evidence edits. The save applies the edits in order and stops at the first refused write:
another writer that changed the same field first answers a conflict and leaves the
rest staged. Edits to different fields of one finding, or to different findings,
do not conflict.
**Discard all** drops every staged edit across all three tabs and restores the
last-saved values.
