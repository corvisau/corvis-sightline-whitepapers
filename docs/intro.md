---
sidebar_position: 1
slug: /
---

# Welcome to the Sightline Knowledge Base

Sightline is a connected modelling platform for OT and industrial cyber
security work. Instead of keeping architecture diagrams, risk registers and
compliance trackers as three disconnected artefacts, Sightline binds them
into one workspace of linked models, so a change in one place is visible
everywhere it matters.

## The Three Pillars

Sightline is organised around a natural pipeline: model the architecture,
ground risk in that architecture, then prove compliance against it.

- **[Tetra](/docs/sightline/architecture/tetra)** models the architecture: zones, devices, networks,
  channels, flows and information, stored as plain YAML rather than a
  drawing that goes stale the day after it's exported.
- **[Lamina](/docs/sightline/architecture/lamina)** models the risk: causes, exposures, controls and
  consequences, using the bow-tie methodology, with entries that can
  reference real elements of a bound Tetra model instead of living as free
  text in a spreadsheet.
- **[Metron](/docs/sightline/architecture/metron)** proves compliance: requirement packages bound to a
  Tetra model, assessed per requirement and entity, with an architectural
  rule engine that can surface a Finding and let a person promote it
  straight into a Lamina Cause or link it to an existing Control.

Alongside them, **[Prism](/docs/prism/architecture/overview)** keeps one current, auditable inventory
of the assets in the estate. It collects asset data from cloud APIs, on-prem
agents and files, merges it by rule, and records the source of every value.
