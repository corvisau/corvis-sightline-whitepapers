Metron is Sightline's compliance assessment app. You load a requirement
package (a framework like AESCSF), bind it to a Tetra model, and assess every
requirement against every in-scope entity. You then track what is not met and
what is being done about it.

This is the entry point. Each feature has its own guide, linked below.

---

## Creating a package

Select **New…** on a folder in the Models tree and choose **Metron: New Requirement Package** to open the new-package form.

| Field | What to enter |
|---|---|
| Location | The folder that receives the package. The form shows the folder you chose as text and cannot change it, so a package cannot be created outside the workspace. The form shows the package it will create in this folder as you type the name |
| Name | The package label, as free text on one line |
| Id | The package id. It names the package. The Id fills in from the Name as you type, until you edit it. It has at most 64 characters: lowercase letters, digits and single hyphens. It must differ from the id of every other model in the workspace, and it does not change after Metron creates the package |
| Description | Optional text for the package |

Metron creates the package directly in the Location, with no folder of its own, and adds a starter requirement.

A requirement package authors no rule and no scheme of its own. Its **Packages**
tab starts empty, and the rule and scheme packages it uses are packages you create
or import. Read the [Component packages](/docs/sightline/usage/metron/component-packages) guide for how an
entry on the Packages tab resolves.

A new package opens on a built-in scheme with a Pass outcome and a Fail outcome.
To use your own outcomes, add a scheme package on the **Packages** tab. Then select
its scheme in the **Scheme** picker on the **Package meta** tab.

One folder holds one Metron package, and it can also hold a Tetra model or a Lamina
model. Metron creates nothing when the Location already holds a Metron package, or a
file that the new package would need. Files that are not part of a model do not stop
the create. Each file of a new package is named after its id, for example
`my-pack.metron.yaml` and `my-pack.req.yaml`. Sightline cannot move or delete a model
whose folder holds another model. See
[Rename, move and delete models and folders](/docs/sightline/usage/shared/manage-models-and-folders#when-sightline-refuses).

A workspace often holds several packages, one per site or project. Select **New…**
on the folder the package belongs in. To make a folder for it, use **New folder…**
in the web shell, or create the folder in your file explorer.

## Packages and assessments outside the workspace

Metron opens only a package or an assessment that the Models tree lists. A file from outside the workspace folders does not open, and Metron shows a message that names the file. Add the folder that holds the file to the workspace, then open the file from the tree.

## Requirement ids and labels

Every requirement has two identifiers:

| Field | What it is | Editable? |
|---|---|---|
| `id` | A system id Metron mints on create (`R-XXXXXX`). Every reference to the requirement (a pair or a finding) uses this value | No, fixed for the requirement's lifetime |
| `label` | The human-facing code you read and type, such as `ACM-1` or `P.ENDP.00`. Shown everywhere a requirement's name appears (the grid, the worklist, the rollup dashboard, generated reports) | Yes |

The Requirements grid's **+ Add** row shows the minted `id` as read-only text;
type the `label` and title, then **Create**.

## The shape of an assessment

Everything in Metron hangs off one idea: a **pair**.

> A **pair** is one `(requirement × entity)`: the question of whether a
> practice holds for a zone, system or device.

Pairs are not authored by hand. Each requirement declares a scope rule, Metron
runs it across the model, and every entity the rule selects becomes a pair. Add
a zone to the model and the pairs for it appear; narrow a rule and they become
*orphans* rather than being deleted.
The Orphans grid has its own `[Edit columns]` picker and persisted column
widths, like every other grid in Metron.

Each pair carries two independent axes:

- **Status**: how far the assessment has progressed (draft → in progress → done)
- **Outcome**: what you found (the package's own labels, mapping onto a fixed
  set of fundamentals)

They are orthogonal: a pair can be `done` with a failing outcome. See the
[Assessment overlay model](/docs/sightline/usage/metron/assessment-overlay-model) guide for why Metron
behaves this way.

## The lifecycle

```
requirement × entity   ->  pair
        pair + outcome ->  assessed
     failing outcome   ->  Finding      (a human assertion: "this is not met")
        finding gaps   ->  Coverage     (what gaps are still open)
```

Each arrow is a deliberate human step. Metron does not promote an outcome
into a finding or resolve a finding automatically, and it derives compliance only
from the outcome you picked.

| Step | What it is | Guide |
|---|---|---|
| Assess a pair | Pick an outcome, attach evidence and notes | [Assessment overlay model](/docs/sightline/usage/metron/assessment-overlay-model) |
| Work faster | Multi-select, inline cells, bulk edit | [Bulk edit](/docs/sightline/usage/shared/bulk-edit) |
| Copy one assessment onto many pairs | Multi-select, then bulk edit (a one-time copy) | [Bulk edit](/docs/sightline/usage/shared/bulk-edit) |
| Keep many pairs on one assessment | A shared assessment that linked pairs track live | [Shared assessments](/docs/sightline/usage/metron/shared-assessments) |
| Cite shared evidence | One reference, many pairs | [General (Assessment-Level) Evidence](/docs/sightline/usage/metron/general-evidence) |
| Record sub-characteristics | Per-attribute target vs current state | [Per-attribute assessment](/docs/sightline/usage/metron/attribute-assessment) |
| Raise a finding | Promote failing outcomes | [Findings Tab](/docs/sightline/usage/metron/findings) |
| Track the work | The coverage join over findings | [Coverage](/docs/sightline/usage/metron/coverage) |
| Narrow what you see | Worklist filters and slices | [Worklist Filtering, Grouping, and Slices](/docs/sightline/usage/metron/worklist-filtering) · [Requirements Filtering and Column Visibility](/docs/sightline/usage/metron/requirements-filtering) |
| Automate outcomes | Bonsai query rules | [Rules](/docs/sightline/usage/metron/rules) |
| Share rules, attribute sets or schemes across packages | Standalone component packages, added on the Packages tab | [Component packages](/docs/sightline/usage/metron/component-packages) |
| Bring in outside data | Audit exports, attestation feeds | [External data](/docs/sightline/usage/metron/external-data) |

Also here: [Licensing](/docs/sightline/usage/shared/licensing) (what edit-gating applies) and
[Telemetry](/docs/sightline/usage/shared/telemetry).

## Two mechanics worth understanding early

Both behaviours are deliberate.

### 1. References are version-pinned, and that is the point

A finding does not point at a live pair. It records a copy, stamped with
the requirement and entity versions it was raised against, plus the outcome at
that moment.

That is what makes a finding an **assertion** rather than a live query. It also
means Metron has to tell you when the model has changed under a finding. It does
this with a **Stale** signal rather than by quietly rewriting anything.

### 2. Formal record vs live signal

Metron holds two kinds of state:

- A finding, and any generated docx or pptx document, is the *formal record*: what
  a human has asserted. If nobody has revised it, the previous formal state
  still holds, and a document that still lists the gap is correct.
- The app additionally shows live state (what is true right now), so you
  can decide what needs revising.

This is why the Coverage tab's `Attribute met` column exists in the app but
is deliberately absent from generated documents. See the "Formal record vs live
signal" section of the [Assessment overlay model](/docs/sightline/usage/metron/assessment-overlay-model)
guide for the full reasoning.

## Nothing is filtered out for you

Every Metron report shows everything and gives you the controls to narrow it.
Resolved findings and met attributes all stay visible, with their state as
filterable columns.
