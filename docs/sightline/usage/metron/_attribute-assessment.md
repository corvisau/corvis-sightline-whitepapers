A requirement's attributes say what the target state is. Assessing a
practice records the current state of each one. The gap is then readable per
attribute instead of hiding inside a single outcome plus free text.

```
Target State   deployed   owned by SCADA team, across every OT zone
Current State  deployed   partial - 2 of 5 zones
```

---

## Which attributes are assessable

**An attribute is assessable when it has a type.** An attribute without
one is descriptive (a maturity level or a progression area). It has no current
state to find, so it never appears in the drawer.

The five types are Indicator, Activity, Standard, Assurance and Governance. You choose the type in the requirement drawer, or in the attribute drawer of an attribute set. See [Attributes](/docs/sightline/usage/metron/attributes) for the attribute model itself
and the four-layer resolution order.

A requirement gets an attribute set only when you attach it. See
[Authoring attributes and attribute sets](/docs/sightline/usage/metron/attribute-authoring).

## Assessing

The Assess drawer of a practice shows a **Target State** section (only when
at least one assessable attribute resolves) below Outcome and Evidence. One row
per attribute: its label and a state control, with the declared target (value,
owner, description) beneath.

| State | Meaning |
|---|---|
| Met | The target is in place. |
| Partial | Partly in place. The sometimes-done case a checkbox could not express. |
| Not met | Not in place. |
| *Not assessed* | Nobody has looked yet. This is the absence of a record, not a fourth value. |

Select **Not assessed** to delete the entry. Type a note on an attribute nobody
has assessed to create it as **Not met**: recording detail means you looked and
found something wanting.

The section header counts **Met** against the total (`2/5 met`). The drawer marks
every row that is not **Met** as a gap, unassessed rows included.

Edits stay unsaved until you select the drawer's Save button, together with the
rest of the pair's edits.

**Per-attribute state is not bulk-editable.** The worklist's Bulk Edit dialog
stays limited to Outcome/State/Notes. Each practice is assessed independently,
and progression-group assessment relies on that (see
[worklist filtering](/docs/sightline/usage/metron/worklist-filtering)). A bulk edit would apply one
practice's findings to many selected pairs.

## It never derives the outcome

Attribute state never maps onto the outcomes of the assessment scheme, and it
does not change the outcome or the state of a pair.
It does not appear in the Dashboard rollup distribution or CSV export.

A practice keeps the single outcome it always had. The assessor chooses it in
the outcome picker. The attributes inform that judgement.

## Persistence and retention

The Assess drawer stores the assessment of each attribute on the pair. The pair owns these assessments and never inherits them from an attribute set. An attribute with no assessment has no stored entry.

An assessment outlives its attribute. If you remove the attribute or detach its set, the row leaves the drawer. If the attribute returns with the same name, the work of the assessor reappears.

## Promoting a pair

Promoting a pair to a finding adds the whole pair as a member of the finding. The
attribute states stay on the pair; a finding does not copy them. See
[Findings Tab](/docs/sightline/usage/metron/findings) for members and their link states.

The [Coverage](/docs/sightline/usage/metron/coverage) tab shows the member as a whole-practice row, and the
**Attribute met** column stays blank for it.

## Related

- [Attributes, attribute sets and tags](/docs/sightline/usage/metron/attributes): the attribute model
  and the four-layer resolution order.
- [Assessment overlay model](/docs/sightline/usage/metron/assessment-overlay-model): outcomes, statuses
  and the formal-record-vs-live-signal separation this respects.
- [Worklist filtering](/docs/sightline/usage/metron/worklist-filtering): Compliance slices and the
  Basic/Advanced filter tabs.
