The **Findings** tab lists every authored finding for the assessment. A finding
is created by promoting one or more flagged worklist outcomes (see the
worklist's **Promote to finding** button on
[the worklist filtering page](/docs/sightline/usage/metron/worklist-filtering)). Every field is directly
editable in the grid, and the tab shares the same toolbar mechanism as the
Assessment Worklist tab.

## Fields

| Column | Editable as | Notes |
|---|---|---|
| Label | text | Short finding title. |
| Description | multi-line text | Free-form detail. |
| Status | select (open / reviewed / resolved) | The human review lifecycle. |
| Severity | select (low / medium / high / critical) | Author-assigned; defaults to `medium` when absent (findings authored before this field existed). |
| Tags | chip add/remove | Free-text labels; type a tag and press Enter to add, click a chip's `×` to remove. |
| Stale | read-only badge | Shows `stale` when any of the finding's refs point at an entity/outcome that has since drifted. Never editable. |

Click any editable cell to enter edit mode. Text fields stage on blur or
Enter, selects stage on choosing an option, and tags stage on every
add/remove. A staged cell shows a dirty highlight. Nothing writes to
disk until you click **Save all** in the shared save bar, which appears
whenever any Worklist, Findings, or Evidence edit is unsaved. **Discard all**
drops every staged edit across all three tabs.

## Findings registers

A finding lives in a **findings register**: any `*.finding.yaml` file in the
workspace. A register holds a top-level `findings:` array, and the author picks
its folder and its name. One register collects findings across assessments,
models and requirement packages, which is the shape a findings register outside
the workspace expects.

The promote dialog asks which register to use whenever you create a finding.

| The workspace has | The dialog shows |
|---|---|
| No register yet | A **Register folder** field reading `findings` and a **Register file name** field reading `authored.finding.yaml` |
| One or more registers | A **Register** picker, preselected to the register this assessment last wrote to, with a **New register...** entry that reveals the folder and name fields |

Metron reads every register in the workspace. It never reads one inside
`node_modules`, `.git`, `dist`, `out` or `.worktrees`.

Adding outcomes to an existing finding writes to that finding's own register, so
the promote dialog hides the register control in **Add to existing** mode.

A register name ends `.finding.yaml`, and its path stays inside the workspace.
The dialog appends the suffix to a name that omits it.

## What a promoted reference cites

Promoting a pair attaches every attribute the assessor recorded as **Not met**
or **Partial** on it. A pair with two unmet attributes therefore gives two
independently trackable gaps. A pair with nothing recorded, or with everything
**Met**, promotes as a single whole-practice reference. See the section *What
promoting a pair attaches* on the
[Per-attribute assessment](/docs/sightline/usage/metron/attribute-assessment) page.

Those attribute names are stored on the reference as `attributeNames`. A file
still carrying the pre-rename `indicatorKeys` loads, but raises a warning in
the Problems panel and loses the key on the next save. Rename the key in
place to keep the values, which are already bare attribute names.

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

## Saving

Every field above stages into one pool of unsaved changes shared with the Worklist and
Evidence tabs. **Save all** flushes every staged finding in a single batched
write, together with any other staged Worklist or Evidence edits. The write
groups findings by the `*.finding.yaml` file each one lives in, so a save
rewrites each touched file once. **Discard all** drops every staged edit across
all three tabs and restores the last-saved values.
