Sightline is a VS Code extension for modelling architecture, risk and compliance. Each model is a set of plain YAML files on disk. The extension draws interactive views of those files and writes your edits back to them.

## The apps

| App | What it does |
|---|---|
| [Tetra](/docs/sightline/architecture/tetra) | Models networks and systems as block diagrams |
| [Lamina](/docs/sightline/architecture/lamina) | Models risk as bow-tie diagrams |
| [Metron](/docs/sightline/architecture/metron) | Assesses a model against a set of requirements |

A YAML file that none of the apps claims opens as plain text.

## What the apps share

- **Plain YAML models.** You can review, compare and version the files with any tool that reads text. A model directory holds the files of one model.
- **The same editing surfaces.** Lamina and Tetra draw a canvas and show the same data in a Tabular Editor. Both open a props drawer for one entity at a time.
- **Data slices.** A data slice is a saved, named view that narrows a model with a filter expression.
- **One authoring language for templates.** Lamina and Tetra resolve templates and partials with the same engine.
- **One licence.** A licence unlocks editing for each app that it names, and one licence covers every app on a machine. See [Licensing](/docs/sightline/usage/shared/licensing).

## How the apps work together

The apps find each other's files in the workspace. A finding that Metron records can become a cause in Lamina. Tetra can draw the assessment coverage of Metron on its diagram. See [Cross-app workflows](/docs/sightline/architecture/cross-app-workflows).

## Where to go next

- [Install Sightline](/docs/sightline/install/extensions)
- [Quickstart](/docs/sightline/usage/quickstart)
