A Metron assessment applies a requirement package to a Tetra model. Each requirement selects the entities it applies to, and each requirement and entity together make one pair. A person assesses each pair, and raises findings over the pairs that need action. Each part of an assessment has one job, and an assessment stays readable when each part does only that job.

## The parts of an assessment

| Part | What it represents | Where it lives |
|---|---|---|
| Requirement package | A set of requirements to assess, such as one framework or one control set | Its own package |
| Requirement | One practice that the package asks for | The requirement package |
| Scope rule | The query that selects the entities a requirement applies to | A rule package |
| Pair | One requirement and one entity, assessed together | The assessment |
| Scheme | The outcomes and the statuses an assessor can choose | A scheme package |
| Assessment rule | A query that suggests an outcome for each pair | A rule package |
| Attribute | A named property of a requirement, which can carry a value, a type and an owner | An attribute set, a requirement or a pair |
| Tag | A free label with no value | A requirement or a finding |
| Shared assessment | One outcome, comment and evidence that many pairs link to | The assessment |
| General evidence | A reference record that many pairs cite | The assessment |
| Finding | A person's statement about a group of pairs, with a status and a severity | The Tetra model |

A requirement package uses rules, attribute sets and a scheme from component packages, and it authors none of them. See [Component packages](/docs/sightline/usage/metron/component-packages).

## Requirements

A requirement states one practice. Write one requirement for each practice that gets its own outcome. Record the separate checks of one practice as typed attributes of its requirement.

A requirement carries an id and a label. Metron sets the id, and every pair and finding uses it, so it does not change. The label is the code that people read, such as `ACM-1`, and you can change it.

The requirement type is `control` or `architecture`. The type describes the requirement and changes no assessment. Use `architecture` for a practice about the shape of the system, such as zone separation, and `control` for any other practice.

The category is a path of group names. The Dashboard tab and the portfolio roll-up count outcomes at each level of that path, so give the category the structure of the framework.

## Scope and pairs

The scope rule of a requirement selects the entities that the requirement applies to. Metron runs the rule over the whole bound model, and each entity that the rule selects becomes a pair. Nobody authors a pair by hand.

Choose the entity kind that holds the practice. A practice that a boundary enforces applies to zones. A practice that each machine must meet applies to devices. A practice about a connection applies to channels or flows. See [Tetra entities](/docs/sightline/methodology/tetra-entities) for what each kind represents.

A pair follows the model. Add a zone and its pairs appear. Narrow the rule, or remove an entity, and its pairs become orphans, which keep their assessment.

## Outcomes and statuses

Each pair carries two independent values. The status records how far the assessment has progressed, and the outcome records what the assessor found. A pair can be done with a failing outcome.

Metron fixes five fundamental outcomes and three fundamental statuses:

| Fundamental outcomes | Fundamental statuses |
|---|---|
| `yes`, `no`, `partial`, `indeterminate`, `not-applicable` | `draft`, `in-progress`, `done` |

The scheme gives each outcome and each status a label in the language of the framework, and maps each label to one fundamental. Several labels can map to one fundamental. Use this to keep reasons apart that the framework treats as different, such as two not-applicable outcomes for out of scope and below threshold. The Dashboard tab, the worklist filters and the exports read the fundamental under each label.

The scheme has no weighted score. The fundamental of the chosen outcome is the compliance result, and the Dashboard tab shows how many done pairs have each fundamental. Choose labels that name the answers an assessor gives.

A requirement can replace the statuses of the scheme with its own list. Use this only when that requirement has a different review workflow. Each list covers all three fundamentals. See [Assessment overlay model](/docs/sightline/usage/metron/assessment-overlay-model).

## Manual and rule assessment

A requirement sets its assessment method to `manual` or `rule`. With `manual`, the assessor picks each outcome. With `rule`, an assessment rule suggests an outcome for each pair it returns.

Use `rule` when the model holds the fact that settles the requirement. A rule that reads whether a zone has cross-zone exposure can suggest an outcome for every zone. Use `manual` when the answer is in documents, interviews or a site visit.

A rule never completes a pair. An outcome that the assessor sets takes precedence over the outcome of the rule, and only a person moves a pair to done. A pair for which the rule returns no row stays unassessed. See [Rules](/docs/sightline/usage/metron/rules).

## Attributes and tags

An attribute is a named property with an optional value, such as `mil` with the value `MIL3`. A typed attribute is also a check that the assessor records against each pair as met, partial or not met. These per-attribute states never change the outcome or the Dashboard count.

A tag is a free label with no value, such as `EDR`. Use an attribute when the property has a value that a filter or a slice compares. Use a tag to mark a requirement for a person, such as work to do later.

Put attributes that many requirements share in an attribute set. A requirement attaches the set and overrides any attribute of the set by name. A pair can override attributes in the same way, which records a target state for one entity without a change to the requirement. See [Attributes, attribute sets and tags](/docs/sightline/usage/metron/attributes).

## Reusing an assessment

Many pairs often get the same answer, such as one control that applies the same way in fifteen zones. Metron has four ways to reuse work across pairs:

| Tool | Choose it when |
|---|---|
| [Shared assessment](/docs/sightline/usage/metron/shared-assessments) | The pairs must keep the same outcome, comment and evidence, and an edit must reach every pair |
| [Bulk edit](/docs/sightline/usage/shared/bulk-edit) | The pairs start from the same values once, and each pair then changes on its own |
| [General evidence](/docs/sightline/usage/metron/general-evidence) | Many pairs cite the same document, but each pair has its own outcome |
| [Attribute set](/docs/sightline/usage/metron/attributes) | Many requirements share the same checks |

A pair that links a shared assessment can still set its own value for any field. Its own value takes precedence over the shared one.

## Findings

A finding is a statement that a person makes about a group of pairs. Each pair in the group is a member. The finding has a label, a status, a severity and tags.

Raise a finding for each problem that needs one response. One finding can group the pairs of several requirements or several entities, when one action resolves all of them. Any done pair that does not need revalidation can become a member, whatever its outcome. A `no` or `partial` outcome usually calls for a finding, and a `yes` outcome can support a positive finding for an audit.

A finding records each member as it was when the member was added. When the outcome, the attribute states, the requirement or the entity of a member changes later, the finding shows as stale. The finding stays the formal record until a person reviews it. Generated documents show the formal record, and the app also shows the live state. See [Findings Tab](/docs/sightline/usage/metron/findings).

A person can promote a finding into a Lamina cause, which links the gap to the risk register. See [Cross-app workflows](/docs/sightline/architecture/cross-app-workflows).

## Component package versions

A requirement package names each component package that it uses. Each entry is pinned to one version or floats to the installed version. Pin an entry when an assessment must stay stable against the rules and the scheme it was assessed with. Let an entry float while the catalogue still changes.

An assessment stays on the version of the requirement package that it was made against. To assess a new version of the package, make a new binding. The portfolio roll-up counts only the assessments of one package version. See [Portfolio roll-up](/docs/sightline/usage/metron/portfolio-rollup).

## Choosing between them

| You want to | Use |
|---|---|
| Assess one practice of a framework | A requirement |
| Apply a requirement to the entities that hold the practice | A scope rule that selects that entity kind |
| Let the model suggest the outcome | The `rule` assessment method with an assessment rule |
| Record an answer from documents or a site visit | The `manual` assessment method |
| Keep two reasons for the same result apart | Two outcome labels that map to one fundamental |
| Give a requirement a property that filters compare | An attribute |
| Mark a requirement for a person | A tag |
| Record checks that many requirements share | An attribute set |
| Keep many pairs on one answer | A shared assessment |
| Copy values onto many pairs once | Bulk edit |
| Cite one document from many pairs | General evidence |
| Record a problem that needs a response | A finding over the affected pairs |
| Keep an assessment stable against its rules | A pinned component package |
