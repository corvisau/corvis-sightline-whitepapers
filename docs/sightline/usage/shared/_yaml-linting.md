The Sightline JSON Schemas let VS Code check a model or package file as you type. The schemas are published in the `schemas/` directory of the `sightline-public` repository. Each app has its own subdirectory: `schemas/tetra/`, `schemas/bowtie/` for Lamina, and `schemas/metron/`.

## Set up

1. Install the Red Hat "YAML" extension (`redhat.vscode-yaml`).
2. Open the `schemas/README.md` file in `sightline-public`.
3. Copy its `yaml.schemas` block into the `settings.json` of your workspace.

The block maps file names to schemas. The tables below list the mappings for each app.

## Files with other names

The Lamina and Tetra loaders read a data file by its top-level keys, so such a file can have any name. Add a modeline to the first line of a file with a different name. The path in a modeline is relative to the YAML file, so adjust the `../` segments.

Lamina and Tetra both use `systems.yaml`, `overlays.yaml` and `annotations.yaml`. The block does not map them. Add a modeline that names the schema of the app that owns the model.

## Tetra

The Tetra schemas check a model file. The block checks these files:

| File name | Schema |
|---|---|
| `*.tetra.yaml` | `blockdiagrammodel.schema.json` |
| `*.tree.yaml` | `treecollection.schema.json` |
| `zones.yaml` | `zones.schema.json` |
| `groups.yaml` | `groups.schema.json` |
| `devices.yaml` | `devices.schema.json` |
| `networks.yaml` | `networks.schema.json` |
| `channels.yaml` | `channels.schema.json` |
| `flows.yaml` | `flows.schema.json` |
| `information.yaml` | `informations.schema.json` |

For `systems.yaml`, `overlays.yaml` and `annotations.yaml`, add a modeline that names the `tetra` schema. This example checks `systems.yaml`:

```yaml
# yaml-language-server: $schema=../../sightline-public/schemas/tetra/systems.schema.json
```

Limits:

- The schema for a `*.tetra.yaml` manifest checks only the `blockdiagram` key. It does not check the keys `files`, `views`, `queries` and `reports`.
- A manifest with no `blockdiagram.meta` shows a missing-`meta` error. Tetra adds `meta` in memory when it loads the file. Add a `meta` block to the file to clear the error.
- The block does not map template files (`*.template.yaml`) or constants blocks.

## Lamina

The Lamina schemas check a model file. The block checks these files:

| File name | Schema |
|---|---|
| `*.bowtie.yaml` | `bowtiemanifest.schema.json` |
| `causes.yaml` | `causes.schema.json` |
| `events.yaml` | `events.schema.json` |
| `outcomes.yaml` | `outcomes.schema.json` |
| `controls.yaml` | `controls.schema.json` |

This example checks a file that holds `vectors`:

```yaml
# yaml-language-server: $schema=../../sightline-public/schemas/bowtie/vectors.schema.json
```

For `systems.yaml`, `overlays.yaml` and `annotations.yaml`, add a modeline that names the `bowtie` schema.

Limits:

- The block does not map template files (`*.template.yaml`).
- The data-file schemas allow extra top-level keys, so a misspelt top-level key is not reported. Only the manifest schema rejects unknown keys.

## Metron

The Metron schemas check a package file. The block checks these files:

| File name | Schema |
|---|---|
| `*.metron.yaml` | `metronpackagemanifest.schema.json` |
| `*.req.yaml` | `requirements.schema.json` |
| `*.rules.yaml` | `rules.schema.json` |
| `*.attributes.yaml` | `attributesets.schema.json` |
| `*.scheme.yaml` | `assessmentscheme.schema.json` |
| `*.assessment.yaml` | `applicationbinding.schema.json` |
| `*.finding.yaml` | `findingfile.schema.json` |
| `*.external.yaml` | `externaldatafile.schema.json` |

One schema, `metronpackagemanifest.schema.json`, covers all four package kinds: requirement, rules, attributes and scheme. A manifest that matches none of them shows an error.

Limits:

- The schemas allow extra top-level keys, so a misspelt top-level key is not reported.
- Metron files that end in `.rules.metron.yaml`, `.attributes.metron.yaml` and `.scheme.metron.yaml` are manifests. The `*.metron.yaml` entry checks them.
