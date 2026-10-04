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

## Authoring custom labels — the assessment scheme

Every requirement package (`*.metron.yaml`) selects an assessment scheme by id
(`schemeRef`). The scheme carries both axes:

```yaml
id: compliance
outcomes:
  - id: pass
    label: Compliant
    fundamental: yes
  - id: fail
    label: Non-compliant
    fundamental: no
  - id: na-out-of-scope
    label: N/A — Out of Scope
    fundamental: not-applicable
  - id: na-below-threshold
    label: N/A — Below Threshold
    fundamental: not-applicable
statuses:
  - id: draft
    label: Draft
    fundamental: draft
  - id: in-progress
    label: In Progress
    fundamental: in-progress
  - id: waiting-review
    label: Waiting for Internal Review
    fundamental: in-progress
  - id: done
    label: Done
    fundamental: done
```

### Outcomes — `outcomes[]`

Every outcome is an ordinary entry with a **required** `fundamental`
field. There is no limit on how many outcomes map to the same fundamental.
This is how a package expresses multiple not-applicable variants
(`na-out-of-scope` and `na-below-threshold` above).

The scheme has no weighted scoring and no rollup method.
`outcomes[].fundamental` is the sole compliance signal: the user picks an
outcome (or a Bonsai query emits one), and that outcome's fundamental is the
compliance result. See the section "Rollup: a distribution, not a score" below
for how the Dashboard tab counts outcomes.

### Status labels — `statuses[]` (optional)

`statuses[]` holds the per-package custom status labels. When a package omits
it, the package gets the 3 built-in fundamental statuses (id equal to the
fundamental, Title Case label).

When present, `statuses[]` must cover all 3 fundamentals at least once.
Several custom statuses can share a fundamental, as outcomes can:
`in-progress` and `waiting-review` above both map to fundamental
`in-progress`.

Both `outcomes[]` and `statuses[]` reject duplicate `id`s.

## Authoring in the scheme editor

⛔ A scheme is edited in a scheme package, never in a requirement package. The
requirement package's **Scoring scheme** tab renders read-only and names where
the scheme lives. See the [Component packages](/docs/sightline/usage/metron/component-packages) page
for how to create a scheme package and select one of its schemes.

Each outcome row in the Scheme Editor shows an **id**, a **label** and a
**Fundamental** dropdown with 5 options. A **Customize statuses** toggle
reveals a second, separately editable list. The first time you turn it on, the
editor seeds the 3 built-in statuses as a starting point to rename or extend.
Turn it off to clear the override back to the built-in defaults.

## Assessing a pair

The **Outcome Picker** and **Status Picker** are dropdowns. Every outcome,
including each not-applicable variant, is an ordinary entry.

The Status Picker's dropdown lists every status the scheme defines (or the 3
built-in defaults). A `done`-fundamental status renders as a disabled option
until an outcome is chosen. If the pair is currently `done`-fundamental, any
status you pick opens a confirm dialog first.

## Worklist "State" vs "Outcome" - two orthogonal axes

The assessment worklist has two columns that read alike but measure different
things:

| Column | What it shows | Source |
|--------|--------------|--------|
| **State** | Workflow progress: is the pair still being assessed, or finished? | The pair's `status` field mapped through `statuses[]` to a fundamental (`draft` / `in-progress` / `done`) |
| **Outcome** | Compliance result: what did the assessment find? | The pair's `outcome` field mapped through `outcomes[]` to a fundamental (`yes` / `no` / `partial` / `indeterminate` / `not-applicable`) |

The two axes are independent. A pair can be `done` (workflow complete) with
any outcome, or still `draft`/`in-progress` with an outcome already picked (a
Bonsai-assessed pair, for example, may have an `outcome` before the pair is
marked `done`). The fundamental outcome the user or a Bonsai query picked is
the canonical compliance signal, and nothing computes a pass/fail verdict from
it.

Metron allows any `done`, non-revalidating pair with an
assigned outcome to be promoted to a finding, whichever fundamental the outcome
resolves to. A `no` or `partial` outcome suggests that a finding is warranted
but does not restrict promotion. A user can promote a `yes` outcome too, for
example to record a positive finding for audit purposes.

The **Filter** popover's **Outcome** field (see [Worklist Filtering, Grouping, and Slices](/docs/sightline/usage/metron/worklist-filtering)) lets you
narrow the worklist to specific fundamental outcomes directly, for example
`Outcome = no OR Outcome = partial` for everything that needs attention.

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
describes. A pair's `attributeAssessments` never resolves to a
`fundamentalOutcome` or `fundamentalState`, never appears in the Dashboard
rollup distribution above, and never affects promote-eligibility.

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

Resolution is tracked at two grains:

- **Pair grain**: the outcome-changed check fires
  when a pair's live outcome differs from what was stamped on the ref at
  promote time. The Findings tab shows it as the **Stale** column. It appears
  only in the app and never reaches the report envelope or a generated
  document, for the same formal-record reason as attribute satisfaction. A
  consumer can derive it from the envelope.
- **Attribute grain**: whether one specific flagged attribute was met, which
  the pair's outcome cannot answer. The Coverage tab shows it as the
  **Attribute met** column.

## Where the fundamental values show up

- **Worklist state pill**: shows the pair's custom status label as text. The
  pill's `data-state` attribute and colour follow the fundamental value, a
  stable hook independent of which custom label a package uses.
- **Worklist Reval column**: a `⚠` indicator, visible by default, on any
  `done` pair whose assessed requirement or entity version has drifted.
  It is the same `requiresRevalidation` flag that
  promotion checks. Right-click a drifted pair (single or
  bulk-selected) for a **Revalidate** action. The action rolls the
  assessed-version stamps forward to the current live version, behind a
  confirm dialog, and leaves the outcome, evidence and notes untouched. The
  **Filter** popover's **Needs revalidation** condition narrows the worklist
  to flagged pairs (see [Worklist Filtering, Grouping, and Slices](/docs/sightline/usage/metron/worklist-filtering)).
  A pair linked to a [shared assessment](/docs/sightline/usage/metron/shared-assessments) is also
  flagged when its stamped shared version falls behind the record's version.
- **Coverage bar and state filter chips**: 4 buckets (`draft`, `in-progress`,
  `done` and `n/a`), assigned by fundamental. A pair in a custom status mapped
  to `in-progress` counts in the same bucket as a pair with the plain "In
  Progress" status. "Waiting for Internal Review" is an example of such a
  custom status.
- **CSV export**: includes `state_fundamental` and `outcome_fundamental`
  columns alongside the raw `state` and `outcome` columns. A spreadsheet can
  filter or pivot on the shared fundamental axis across packages with
  different custom labels.
- **Report envelope** (pull API, contract v6): every pair in
  `assessment.pairs[]` carries `fundamentalState` and `fundamentalOutcome`
  alongside the raw `state` and `outcome`. The embedded scheme carries no
  `kind`, `rollup`, `outcomes[].weight` or `outcomes[].excludeFromScore`.
- **Finding staleness**: a promoted finding stamps the `fundamentalOutcome`
  assessed at promote time. Metron
  recomputes staleness when the live pair's fundamental outcome no longer
  matches the stamp, or when the live entity version advances past the ref's
  pinned version. The Findings tab shows a **Roll forward** action on
  stale-badged rows. The action accepts the current live version and outcome
  as the new baseline for each affected ref. It runs behind a confirm dialog
  and leaves the finding's review status unchanged. Roll forward leaves untouched a ref whose
  entity no longer resolves in the live model (deleted or re-scoped).
