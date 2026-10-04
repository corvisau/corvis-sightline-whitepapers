A requirement's attributes say what the target state is. Assessing a
practice records the current state of each one. The gap is then readable per
attribute instead of hiding inside a single outcome plus free text.

```
Target State   deployed   owned by SCADA team, across every OT zone
Current State  deployed   partial - 2 of 5 zones
```

---

## Which attributes are assessable

**An attribute is assessable when it declares a `type`.** An attribute without
one is descriptive (a maturity level or a progression area). It has no current
state to find, so it never appears in the drawer.

```yaml
attributeSets:
  - id: mil3-management
    name: MIL3 management characteristics
    attributes:
      documented:                 # assessable - it has a type
        type: indicator
        label: Documented
        owner: SCADA team
        description: The practice is documented in policy or procedure.
      approved:
        type: indicator
        label: Approved by management
```

```yaml
requirements:
  - id: ASSET-1f
    # ...
    attributeSetRefs:
      - mil3-management
    attributes:
      mil:
        value: MIL3               # NOT assessable - no type
```

The five types are `indicator`, `activity`, `standard`, `assurance` and
`governance`. See [Attributes](/docs/sightline/usage/metron/attributes) for the attribute model itself
and the four-layer resolution order.

Attributes reach a requirement by explicit reference: the requirement names the
set in `attributeSetRefs`. No trigger predicate attaches a set implicitly.

## Assessing

The Assess drawer of a practice shows a **Target State** section (only when
at least one assessable attribute resolves) below Outcome and Evidence. One row
per attribute: its label and a state control, with the declared target (value,
owner, description) beneath.

| State | Meaning |
|---|---|
| `met` | The target is in place. |
| `partial` | Partly in place. The "sometimes done" case a checkbox could not express. |
| `not-met` | Not in place. |
| *Not assessed* | Nobody has looked yet. This is the absence of a record, not a fourth value. |

Select **Not assessed** to delete the entry. Type a note on an attribute nobody
has assessed to create it as `not-met`: recording detail means you looked and
found something wanting.

The section header counts `met` against the total (`2/5 met`). The drawer marks
every row that is not `met` as a gap, unassessed rows included.

Edits stay unsaved until you select the drawer's Save button, together with the
rest of the pair's edits.

**Per-attribute state is not bulk-editable.** The worklist's Bulk Edit dialog
stays limited to Outcome/State/Notes. Each practice is assessed independently,
and progression-group assessment relies on that (see
[worklist filtering](/docs/sightline/usage/metron/worklist-filtering)). A bulk edit would apply one
practice's findings to many selected pairs.

## It never derives the outcome

Attribute state never maps onto `AssessmentScheme`'s fundamental outcome
vocabulary or contributes to a pair's `fundamentalOutcome` / `fundamentalState`.
It does not appear in the Dashboard rollup distribution or CSV export.

A practice keeps the single outcome it always had. The assessor chooses it in
the outcome picker. The attributes inform that judgement and never compute it.

## Persistence

A pair's assessments live under `attributeAssessments` on the pair, keyed by the
bare attribute name. At the attribute layer a name collision is how overriding
works:

```yaml
pairs:
  - requirementId: ASSET-1f
    entityId: z-ot
    state: in-progress
    outcome: null
    evidence: []

    # What this pair says the TARGET is (inheritable, four-layer).
    attributeSetRefs: [target-2027]
    attributes:
      documented:
        owner: SCADA team

    # What the assessor FOUND (owned outright, never inherited).
    attributeAssessments:
      documented:
        state: partial
        note: policy drafted, not yet approved
      approved:
        state: not-met
```

The field is absent (not `{}`) when nothing is assessed, matching the
`evidenceRefs` convention.

### `attributes` and `attributeAssessments` are opposites

The two fields share a key space and nothing else:

| | `attributes` | `attributeAssessments` |
|---|---|---|
| Says | the target | what was found |
| Inherited from attached sets | yes, four layers | never |
| Written back | only what the pair itself declares | verbatim |

## Retention — an assessment outlives its attribute

When an attribute is removed from a set, or the set is detached, no pair's
already-recorded `attributeAssessments` change. The row stops appearing in the
drawer on the next resolve. The orphaned entry stays in `*.assessment.yaml`,
inert. If the attribute returns with the same name, the assessor's work
reappears intact. Otherwise the entry stays until someone cleans it up by hand.

## What promoting a pair attaches

Promoting a pair to a finding attaches its unmet attributes to the resulting
reference. Remediation can then be tracked one attribute at a time rather than
one practice at a time.

| Recorded state | Attached as a gap? |
|---|---|
| **Not met** | Yes |
| **Partial** | Yes — a gap is closed when the target is met, not approached |
| **Met** | No |
| *Not assessed* | No — the absence of an entry means nobody looked, which is not something the assessor is asserting |

If nothing on the pair qualifies (every attribute **Met**, or none assessed at
all), the reference is attached at whole-practice granularity instead. Metron still attaches an attribute recorded against a name that no
longer resolves. It carries orphaned references rather than dropping them (see
the Retention section above), and the coverage report shows the raw name.

The attribute names live on the reference as `attributeNames`. From there they
drive the Findings drawer's per-attribute checkboxes, the Coverage tab's
**Attribute** and **Attribute met** columns, and the report envelope's
`metron.coverage` rows. See the [Coverage](/docs/sightline/usage/metron/coverage) page for the
coverage report.

## Related

- [Attributes, attribute sets and tags](/docs/sightline/usage/metron/attributes): the attribute model
  and the four-layer resolution order.
- [Assessment overlay model](/docs/sightline/usage/metron/assessment-overlay-model): outcomes, statuses
  and the formal-record-vs-live-signal separation this respects.
- [Worklist filtering](/docs/sightline/usage/metron/worklist-filtering): Compliance slices and the
  Basic/Advanced filter tabs.
