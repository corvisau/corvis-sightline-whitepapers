Every assessed `(requirement × entity)` pair carries two axes: a **lifecycle
status** (is it done being assessed?) and an **outcome** (what did the
assessment find?). Both axes work the same way: a **fixed, cross-package
fundamental vocabulary** that every package's **custom labels** map onto.
A package can speak in its own domain language ("Waiting for Internal
Review", "N/A — Out of Scope"). Metron's tooling (the rollup dashboard,
worklist filtering, CSV and report export) reasons on the shared fundamental
values underneath, whichever custom labels a package uses.

## The two fundamental vocabularies

### Status — 3 fundamentals

| Fundamental | Meaning |
|---|---|
| `draft` | Not yet worked — editable |
| `in-progress` | Being worked — editable |
| `done` | Finished — read-only until reopened |

### Outcome — 5 fundamentals

| Fundamental | Meaning |
|---|---|
| `yes` | The requirement is met |
| `no` | The requirement is not met |
| `partial` | Partially met |
| `indeterminate` | Assessed, but the answer can't be determined |
| `not-applicable` | Does not apply to this pair |

A package never authors these lists; they are fixed. A package authors its
own **custom labels**, each tagged with the fundamental it maps to.

## Custom labels: the assessment scheme

Every requirement package selects one assessment scheme in the **Scheme** picker on
its **Package meta** tab. The scheme carries both axes: its outcomes and its
statuses. Each outcome and each status has an id, a label and the fundamental it
maps to.

For example, a package can label a `yes` outcome "Compliant" and a `no` outcome
"Non-compliant". It can add two not-applicable variants, "N/A — Out of Scope" and
"N/A — Below Threshold", that both map to `not-applicable`. It can add a status
"Waiting for Internal Review" that maps to `in-progress`.

### Outcomes

Every outcome maps to exactly one of the five fundamental outcomes. Any number of
outcomes can map to the same fundamental. This is how a package expresses
multiple not-applicable variants.

The scheme has no weighted scoring and no rollup method. The fundamental of the
chosen outcome is the sole compliance signal. The user picks an outcome, or an
assessment rule returns one, and that outcome's fundamental is the compliance
result. See the section "Rollup: a distribution, not a score" below for how the
Dashboard tab counts outcomes.

### Status labels

A scheme can define its own status labels. When it defines none, the package gets
the three built-in fundamental statuses, with a Title Case label each.

A custom status list must cover all three fundamentals at least once. Several
custom statuses can share a fundamental, as outcomes can. Within one list, no two
entries share an id.

### Requirement statuses

A requirement can have its own status list. The list replaces the scheme's
statuses for that requirement's pairs. A requirement without one uses the scheme's
list, and a scheme without one uses the three built-in statuses. The same rules
apply: the list covers all three fundamentals and has no repeated id.

A new pair starts in the first status of the list whose fundamental is `draft`.
A rule never moves a pair to a status whose fundamental is `done`.

In the requirement drawer, the **Statuses** field has a **Customize statuses**
toggle. Turn it on to give the requirement its own list, which starts from the 3
built-in statuses, and rename, add or remove rows. Turn it off to return to the
scheme's list. Metron refuses to save a list that does not cover all 3
fundamentals or that repeats an id, and the commit dialog gives the reason. A
pair whose status the list no longer defines reads as Draft until you pick a
status that exists. The State column of the worklist offers the
statuses of every requirement, and a pair takes only a status that its own
requirement defines. A bulk edit that sets a status the requirement of a pair
does not define skips that pair and gives the reason.

## Authoring in the scheme editor

⛔ A scheme is edited in a scheme package, never in a requirement package. The
requirement package's **Scoring scheme** tab renders read-only and names the
scheme package that supplies the scheme. See the [Component packages](/docs/sightline/usage/metron/component-packages) page
for how to create a scheme package and select one of its schemes.

In a scheme package, select **+ Add scheme**, or open a scheme from the Schemes
grid. The scheme drawer lists the outcomes. Each outcome row has an **id**, a
**label** and a **Fundamental** dropdown with 5 options. Select **+ Add outcome**
to add a row. A **Customize statuses** toggle reveals a second, separately
editable list. The first time you turn it on, the editor seeds the 3 built-in
statuses as a starting point to rename or extend. Turn it off to clear the
override back to the built-in defaults.

## Assessing a pair

The **Outcome Picker** and **Status Picker** are dropdowns. Every outcome,
including each not-applicable variant, is an ordinary entry.

The Status Picker's dropdown lists every status the requirement defines, or the scheme defines, or the 3
built-in defaults. A `done`-fundamental status renders as a disabled option
until an outcome is chosen. If the pair is currently `done`-fundamental, any
status you pick opens a confirm dialog first.

## Worklist "State" vs "Outcome" - two orthogonal axes

The assessment worklist has two columns that read alike but measure different
things:

| Column | What it shows | Source |
|--------|--------------|--------|
| **State** | Workflow progress: is the pair still being assessed, or finished? | The pair's status, mapped through the status list to a fundamental (`draft` / `in-progress` / `done`) |
| **Outcome** | Compliance result: what did the assessment find? | The pair's outcome, mapped through the scheme's outcomes to a fundamental (`yes` / `no` / `partial` / `indeterminate` / `not-applicable`) |

The two axes are independent. A pair can be `done` (workflow complete) with
any outcome, or still `draft`/`in-progress` with an outcome already picked. A
Bonsai-assessed pair, for example, may have an outcome before the pair is
marked `done`. The fundamental outcome the user or a Bonsai query picked is
the canonical compliance signal, and nothing computes a pass/fail verdict from
it.

Metron allows any `done`, non-revalidating pair with an
assigned outcome to be promoted to a finding, whichever fundamental the outcome
resolves to. A `no` or `partial` outcome suggests that a finding is warranted
but does not restrict promotion. A user can promote a `yes` outcome too, for
example to record a positive finding for audit purposes.

The **Filter** popover's **Outcome** field (see [Worklist Filtering, Grouping, and Slices](/docs/sightline/usage/metron/worklist-filtering)) lets you
narrow the worklist to specific fundamental outcomes directly. For example,
`Outcome = no OR Outcome = partial` surfaces everything that needs attention.

## Rollup: a distribution, not a score

The Dashboard tab's rollup counts the `done`-fundamental pairs for each
fundamental outcome, at every tree level (requirement, category, package
root). It shows a plain distribution, for example
`yes: 30%, no: 20%, partial: 5%, ...`, and computes no weighted score.
Draft and in-progress pairs are excluded from the count. A pair whose live
requirement or entity has drifted past what was assessed still counts under
its last-assessed outcome.

## Per-attribute state is a separate, unscored axis

Per-attribute assessment (see [Per-attribute assessment](/docs/sightline/usage/metron/attribute-assessment))
records, for each practice, whether each typed attribute is `met`, `partial`
or `not-met`, for example "Documented" or "Approved by management". It is not
a second compliance signal layered on top of the fundamental outcome this page
describes. A pair's attribute states never resolve to a
fundamental outcome or fundamental state. They do not appear in the
Dashboard rollup distribution above, and they do not affect promote-eligibility.

## Formal record vs live signal

Metron carries two kinds of statement, and a report is easy to misread when
the two are confused.

A **finding is a human assertion**, pinned to the requirement and entity
versions it was raised at. It records that, on a given date and against those
versions, the practice was not met. Until a human revises it, that remains the
formal state, so a generated document that still lists the gap is correct
rather than stale.

A **tick in the Assess drawer is live working signal**. It records that someone
has since satisfied one sub-characteristic, and it never revises a finding on
that person's behalf.

Metron surfaces the two differently:

| Surface | Shows | Because |
|---|---|---|
| The app (Coverage tab) | Both - the pinned gap AND an `Attribute met` column reading live state | You are deciding what still needs revising |
| A generated docx/pptx | The formal record only | An assessor reading "satisfied" could reasonably take it as "resolved" when nobody has formally revised anything |

Meeting an attribute never retires a gap row and never contributes to
scoring. Retiring the row would let live working state overwrite a formal
assertion, and would promote attributes into a scored role they do not have.

Resolution is tracked at the pair grain. A member of a finding shows as changed
when the outcome or the attribute states of its pair differ from the pinned
state. The pin is set when the member is added. The Findings tab shows it in the **Stale** column. It appears only in the app
and never reaches a generated document, for the same
formal-record reason as attribute satisfaction.

## Where the fundamental values show up

- **Worklist state pill**: shows the pair's custom status label as text. The
  pill's colour follows the fundamental value, independent of which custom label
  a package uses.
- **Worklist Reval column**: a `⚠` indicator, visible by default, on any
  `done` pair whose assessed requirement or entity version has drifted.
  It is the same `requiresRevalidation` flag that
  promotion checks. Right-click a drifted pair (single or
  bulk-selected) for a **Revalidate** action. The action accepts the
  current requirement, entity and shared assessment, behind a
  confirm dialog, and leaves the outcome, evidence and notes untouched. The
  **Filter** popover's **Needs revalidation** condition narrows the worklist
  to flagged pairs (see [Worklist Filtering, Grouping, and Slices](/docs/sightline/usage/metron/worklist-filtering)).
  A pair linked to a [shared assessment](/docs/sightline/usage/metron/shared-assessments) is also
  flagged when the content of its shared assessment changes. The flag applies
  when that change happens after the pair was completed or last revalidated.
- **Coverage bar and state filter chips**: 4 buckets (`draft`, `in-progress`,
  `done` and `n/a`), assigned by fundamental. A pair in a custom status mapped
  to `in-progress` counts in the same bucket as a pair with the plain "In
  Progress" status. "Waiting for Internal Review" is an example of such a
  custom status.
- **CSV export**: includes `state_fundamental` and `outcome_fundamental`
  columns alongside the `state` and `outcome` columns. A spreadsheet can
  filter or pivot on the shared fundamental axis across packages with
  different custom labels.
- **Report data**: every assessed pair in a generated report carries its
  fundamental state and fundamental outcome beside its custom labels.
- **Finding staleness**: each member of a finding pins the entity version, the
  requirement version and the outcome of its pair when the member is added.
  Metron marks the finding stale when the pair's outcome or either version
  moves past the pin. The Findings tab shows an **Accept** action on
  stale-badged rows. The action pins the current state of every member as the new
  baseline. It runs behind a confirm dialog and leaves the finding's review status
  unchanged. A member whose record no longer exists keeps its stale badge and has
  no pin to move.
