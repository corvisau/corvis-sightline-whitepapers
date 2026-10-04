Metron is the compliance assessment app of Sightline. It binds a requirement package, such as a control set for a security framework, to a Tetra model. It then assesses each requirement against each relevant entity in that model.

## Requirement packages and bindings

A requirement package defines the requirements to assess. When you bind the package to a Tetra model, Metron produces a worklist of pairs, each of one requirement and one entity. Each pair carries its own outcome, evidence and notes.

An outcome stays editable until you mark it complete. When the requirement or the entity changes afterwards, Metron flags the pair as needing revalidation. See [Getting started with Metron](/docs/sightline/usage/metron/getting-started) and [Assessment overlay model](/docs/sightline/usage/metron/assessment-overlay-model).

## Rules and findings

Metron can define rules that run against the bound Tetra model. A rule is a query over the architecture. A requirement names a rule to narrow its scope or to suggest an outcome. See [Rules](/docs/sightline/usage/metron/rules).

A finding is a record that a person creates by promoting assessed pairs. See [Findings](/docs/sightline/usage/metron/findings).

## Closing the loop to risk

A person can promote a finding into a Lamina cause. Metron stores the link on the finding and keeps it even when the architecture changes later. This workflow links an architectural condition in Tetra to a compliance finding in Metron. It also links a finding to an entry in the risk register in Lamina. See [Cross-app workflows](/docs/sightline/architecture/cross-app-workflows).

## Where it fits

Metron needs a bound Tetra model. It feeds Lamina when a person promotes findings. Tetra can draw the coverage of Metron on its diagram. See [Coverage overlay](/docs/sightline/usage/tetra/coverage-overlay).

## Where to go next

- [Getting started with Metron](/docs/sightline/usage/metron/getting-started)
- [Portfolio roll-up](/docs/sightline/usage/metron/portfolio-rollup)
