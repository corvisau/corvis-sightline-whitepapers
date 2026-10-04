## `GET /discovery/models`

Unlike `/models`, this lists every model kind in the workspace, not tetra models only.

### Responses

**200 — The models in the workspace, of every kind.**

```json
[
  {
    "kind": "tetra",
    "id": "plant-a",
    "locator": "example/tetra.yaml"
  },
  {
    "kind": "metron-package",
    "id": "rp-1",
    "locator": "example/pkg.metron.yaml"
  }
]
```

## `GET /discovery/findings`

Reads every findings register in the workspace. A register that fails to parse is skipped rather than failing the whole listing.

### Responses

**200 — Findings, grouped by the register file each came from.**

```json
[
  {
    "sourceFile": "findings/a.finding.yaml",
    "records": [
      {
        "id": "F-1",
        "label": "Weak segmentation"
      }
    ]
  }
]
```

## `GET /discovery/bindings`

Lists every assessment binding candidate found across the workspace's model directories.

### Responses

**200 — Assessment binding candidates.**

```json
[
  {
    "modelId": "plant-a",
    "packageId": "rp-1",
    "packageVersion": 1
  }
]
```

## `GET /discovery/external-data`

Reads every external-data file in the workspace. A file that fails to parse reports its own error alongside the results that did parse.

### Responses

**200 — External data results, one entry per source file.**

```json
[
  {
    "sourceFile": "external/a.external.yaml",
    "tables": [
      {
        "id": "attestations",
        "label": "Control attestations"
      }
    ]
  }
]
```

## `GET /discovery/rules`

packageDir must stay inside the workspace root; a pattern that resolves outside the package directory is rejected the same way.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `packageDir` | query | yes | string | A path to a package directory, confined to the workspace root. |
| `pattern` | query | no | string[] | Repeatable. A glob pattern resolved against packageDir. |

### Responses

**200 — The rules matched by the given patterns.**

```json
[
  {
    "rule": {
      "ruleId": "r-1",
      "title": "Cross-zone exposure"
    },
    "sourceFile": "rules/arch.rules.yaml"
  }
]
```
**400 — packageDir is missing, or packageDir or a pattern resolves outside the package directory.**

```json
{
  "error": "packageDir must not escape the workspace root"
}
```

## `GET /discovery/attribute-sets`

Same parameters and 400 rule as `/discovery/rules`.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `packageDir` | query | yes | string | A path to a package directory, confined to the workspace root. |
| `pattern` | query | no | string[] | Repeatable. A glob pattern resolved against packageDir. |

### Responses

**200 — The attribute sets matched by the given patterns.**

```json
[
  {
    "attributeSet": {
      "id": "s-1",
      "label": "Set 1"
    },
    "sourceFile": "attributes/main.attributes.yaml"
  }
]
```
**400 — packageDir is missing, or packageDir or a pattern resolves outside the package directory.**

```json
{
  "error": "packageDir must not escape the workspace root"
}
```

## `GET /discovery/schemes`

packageDir must stay inside the workspace root and answers 400 otherwise, as for `/discovery/rules`. A pattern that resolves outside the package directory is not rejected here: it comes back as one result with kind "invalid", not a 400.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `packageDir` | query | yes | string | A path to a package directory, confined to the workspace root. |
| `pattern` | query | no | string[] | Repeatable. A glob pattern resolved against packageDir. |

### Responses

**200 — One outcome per matched file: a parsed scheme, an empty file, or an invalid one.**

```json
[
  {
    "sourceFile": "default.scheme.yaml",
    "kind": "scheme"
  }
]
```
**400 — packageDir is missing, or resolves outside the workspace root.**

```json
{
  "error": "packageDir must not escape the workspace root"
}
```

## `GET /discovery/manifest-fragments`

baseDir must stay inside the workspace root; a pattern that resolves outside baseDir is rejected the same way.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `baseDir` | query | yes | string |  |
| `pattern` | query | no | string[] | Repeatable. |

### Responses

**200 — The matched file paths, relative to baseDir.**

```json
[
  "frags/a.yaml",
  "frags/b.yaml"
]
```
**400 — baseDir is missing, or baseDir or a pattern resolves outside baseDir.**

```json
{
  "error": "baseDir must not escape the workspace root"
}
```

## `GET /cross-app/models/{modelId}/findings`

Reads every findings register in the workspace and returns the findings that reference the model, grouped by the entity each reference names. A register that fails to parse is skipped. This is the read that Tetra uses for its findings badges.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The model's own id (`meta.modelId`), the value that findings and assessment bindings carry. It is not the model id used by `/models/{id}`. |

### Responses

**200 — Findings grouped by entity id. An empty array when nothing references the model.**

```json
[
  {
    "entityId": "z-ot",
    "findings": [
      {
        "id": "F-1",
        "label": "Weak segmentation"
      }
    ]
  }
]
```
**403 — The deployment refuses the read. The body carries the reason. Present only when the server is configured with a read check.**

```json
{
  "error": "This licence does not include Metron data in Tetra."
}
```

## `GET /cross-app/models/{modelId}/coverage`

Returns one coverage entry for each entity of the model that an assessment covers, summed across every package that assesses it. Each assessment is checked against the current model, so the counts match the worklist. This is the read that Tetra uses for its coverage overlay.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The model's own id (`meta.modelId`), the value that findings and assessment bindings carry. It is not the model id used by `/models/{id}`. |

### Responses

**200 — Coverage entries sorted by entity id. An empty array when no assessment covers the model.**

```json
[
  {
    "entityId": "z-ot",
    "counts": {
      "draft": 1,
      "inProgress": 0,
      "done": 1,
      "na": 0,
      "requiresRevalidation": 0,
      "total": 2
    },
    "distribution": {
      "yes": 0,
      "no": 0,
      "partial": 1,
      "indeterminate": 0,
      "not-applicable": 0
    },
    "packages": [
      {
        "packageId": "example-package",
        "packageTitle": "Example package",
        "bindingRel": "models/example.assessment.yaml"
      }
    ]
  }
]
```
**403 — The deployment refuses the read. The body carries the reason. Present only when the server is configured with a read check.**

```json
{
  "error": "This licence does not include Metron data in Tetra."
}
```

## `GET /cross-app/models/{modelId}/covering-causes`

Scans every Lamina model in the workspace and returns, for each channel or flow of the Tetra model, the causes whose vector link names one of the model's vectors. A cause whose link names a different Tetra model is left out. A cause appears once for each channel or flow it covers. This is the read that Tetra uses for its Causes section and cause badges.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The Tetra model's own id, the id that `/models` lists for it. It is not the `meta.modelId` that the findings and coverage reads take. |

### Responses

**200 — The covering causes grouped by channel or flow id. An empty array when no cause links a vector of the model.**

```json
[
  {
    "relationId": "c-ot-it",
    "causes": [
      {
        "laminaModelId": "example-risk-model",
        "laminaModelName": "Example risk model",
        "causeId": "C-001",
        "causeLabel": "Remote access abused",
        "vectorKey": "c-ot-it>z-it"
      }
    ]
  }
]
```
**404 — No Tetra model has this id.**

```json
{
  "error": "model not found: example-grid"
}
```
**409 — More than one Tetra model has this id.**

```json
{
  "error": "model id example-grid names 2 models",
  "reason": "ambiguous",
  "candidates": [
    {
      "kind": "tetra",
      "id": "example-grid"
    },
    {
      "kind": "tetra",
      "id": "example-grid"
    }
  ]
}
```

## `GET /cross-app/models/{modelId}/vectors`

Loads the Tetra model and returns the Tetra Vectors it derives: one for each channel or flow that crosses into a zone. This is the read behind a Lamina cause's vector picker and vector links.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The Tetra model's own id, the id that `/models` lists for it. It is not the Lamina model's id. |

### Responses

**200 — The model's vectors. An empty array when no relation crosses into a zone.**

```json
[
  {
    "key": "c-ot-it>z-it",
    "relationId": "c-ot-it",
    "relationKind": "channel",
    "label": "Remote access link",
    "nodeA": "d-ot-1",
    "nodeB": "d-it-1",
    "zoneId": "z-it",
    "otherZoneId": "z-ot",
    "ruleIds": [
      "r-remote-access"
    ],
    "suggestedControls": []
  }
]
```
**404 — No Tetra model has this id.**

```json
{
  "error": "model not found: example-grid"
}
```
**409 — More than one Tetra model has this id.**

```json
{
  "error": "model id example-grid names 2 models",
  "reason": "ambiguous",
  "candidates": [
    {
      "kind": "tetra",
      "id": "example-grid"
    },
    {
      "kind": "tetra",
      "id": "example-grid"
    }
  ]
}
```

## `GET /cross-app/models/{modelId}/vector-staleness`

Compares the Tetra Vector snapshot that each cause of the Lamina model holds with the live vector it links to, in the Tetra models under the Lamina model's folder. `current` means the live vector matches the snapshot. `changed` means it differs, and `changes` names each differing field. `gone` means the key is no longer derived, or the named Tetra model is not loaded, and `reason` says which. A cause with no vector link or snapshot is not listed.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The Lamina model's own id (`meta.id` of its `*.bowtie.yaml`). It is not a Tetra model's id. |

### Responses

**200 — One entry for each cause that links a Tetra Vector with a snapshot. An empty array when none does.**

```json
[
  {
    "causeId": "C-001",
    "key": "c-ot-it>z-it",
    "modelId": "example-grid",
    "state": "changed",
    "changes": [
      {
        "field": "label",
        "old": "Remote access link",
        "new": "Remote access path"
      }
    ]
  }
]
```
**404 — No Lamina model has this id.**

```json
{
  "error": "model not found: example-grid"
}
```
**409 — More than one Lamina model has this id.**

```json
{
  "error": "model id example-grid names 2 models",
  "reason": "ambiguous",
  "candidates": [
    {
      "kind": "bowtie",
      "id": "example-grid"
    },
    {
      "kind": "bowtie",
      "id": "example-grid"
    }
  ]
}
```

## `GET /cross-app/models/{modelId}/vector-coverage`

Lists the Tetra Vectors in the zones that the Lamina manifest declares under `tetraZones`, in the Tetra models under its folder, that no cause links. A manifest that names a `tetraModel` limits the scan to that model. `state` is `no-tetra` when no Tetra model sits under the folder, `model-not-found` when the named model is absent, `no-scope` when the manifest declares no zones, and `ready` otherwise. `total` counts the vectors in the declared zones, covered or not.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The Lamina model's own id (`meta.id` of its `*.bowtie.yaml`). It is not a Tetra model's id. |

### Responses

**200 — The coverage of the Lamina model.**

```json
{
  "state": "ready",
  "zones": [
    {
      "zoneId": "z-it",
      "vectors": [
        {
          "key": "c-ot-it>z-it",
          "relationId": "c-ot-it",
          "relationKind": "channel",
          "label": "Remote access link",
          "nodeA": "d-ot-1",
          "nodeB": "d-it-1",
          "zoneId": "z-it",
          "otherZoneId": "z-ot",
          "ruleIds": [
            "r-remote-access"
          ],
          "suggestedControls": [],
          "modelId": "example-grid"
        }
      ]
    }
  ],
  "total": 3,
  "unknownZones": []
}
```
**404 — No Lamina model has this id.**

```json
{
  "error": "model not found: example-grid"
}
```
**409 — More than one Lamina model has this id.**

```json
{
  "error": "model id example-grid names 2 models",
  "reason": "ambiguous",
  "candidates": [
    {
      "kind": "bowtie",
      "id": "example-grid"
    },
    {
      "kind": "bowtie",
      "id": "example-grid"
    }
  ]
}
```

## `GET /metron/assessments/{id}`

Returns the parsed assessment and a version. The version is a hash of the assessment file's text. Send it back as `baseVersion` when you post edits to `/metron/assessments/{id}/ops`.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string | The assessment's own id, the `id` that its `.assessment.yaml` file carries. It is not a model id from `/models`. |

### Responses

**200 — The assessment and its version.**

```json
{
  "version": "4be1...",
  "binding": {
    "id": "as-pump",
    "meta": {
      "model": "blockdiagram.tetra.yaml",
      "modelId": "bd-plant-a",
      "packageId": "pkg-grid",
      "packageVersion": 1,
      "assessor": "assessor@example.test",
      "startedAt": "2026-06-04T00:00:00.000Z"
    },
    "pairs": [],
    "log": []
  }
}
```
**403 — The deployment refuses the read. The body carries the reason. Present only when the server is configured with a read check.**

```json
{
  "error": "This licence does not include Metron data."
}
```
**404 — No assessment has this id.**

```json
{
  "error": "assessment not found: as-pump"
}
```
**409 — The id names more than one assessment. `reason` is `ambiguous` and `candidates` lists them.**

```json
{
  "error": "assessment id as-pump names 2 assessments",
  "reason": "ambiguous",
  "candidates": [
    {
      "kind": "metron-assessment",
      "id": "as-pump",
      "locator": "plant-a/pump.assessment.yaml"
    }
  ]
}
```

## `POST /metron/assessments/{id}/ops`

Applies the edits in `ops` in order and writes the assessment file once. The batch applies only when `baseVersion` matches the current version, so a stale write answers 409 with the current version instead of overwriting a newer edit. If any edit fails, nothing is written. The server stamps `by` and `at` on every log entry it writes, from the assessment's assessor and its own clock. A request cannot set them. The edit kinds are `applyAssessment`, `setPairState`, `revalidatePairs`, `reconcileBinding`, `archiveOrphan`, `setGeneralEvidence`, `setSharedAssessments`, `saveSlice` and `deleteSlice`.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string | The assessment's own id, the `id` that its `.assessment.yaml` file carries. It is not a model id from `/models`. |

### Responses

**200 — The batch applied.**

```json
{
  "ok": true,
  "version": "c77a...",
  "filesChanged": [
    "plant-a/pump.assessment.yaml"
  ]
}
```
**400 — The body is malformed, an edit is invalid, or an edit failed. Nothing was written. `error` says which.**

```json
{
  "ok": false,
  "reason": "invalid",
  "error": "metronArchiveOrphan: no pair R9/E9 to archive"
}
```
**403 — The deployment refuses the write, or the file could not be written. Present when the server is configured with a write check.**

```json
{
  "ok": false,
  "reason": "forbidden",
  "error": "This deployment is read-only."
}
```
**404 — No assessment has this id.**

```json
{
  "error": "assessment not found: as-pump"
}
```
**409 — `baseVersion` is not the current version, with `reason` set to `conflict`. Also answers 409 with `reason` set to `ambiguous` when the id names more than one assessment.**

```json
{
  "ok": false,
  "reason": "conflict",
  "current": "c77a...",
  "paths": [
    "plant-a/pump.assessment.yaml"
  ]
}
```

## `POST /metron/assessments/{id}/run`

Assesses every pair of a rule-assessed requirement that has no outcome yet, and writes the assessment file once. Each outcome lands as a draft. Only a requirement that opts in to automatic completion is also completed, and the log attributes that to the rule. Pairs that already hold an outcome are never changed. The request body is optional. When it carries `baseVersion`, the run applies only if that matches the current version, and a stale run answers 409 without writing. A run that changes nothing answers 200 with `filesChanged` empty and the current version.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string | The assessment's own id, the `id` that its `.assessment.yaml` file carries. It is not a model id from `/models`. |

### Responses

**200 — The run finished.**

```json
{
  "ok": true,
  "version": "9c2d...",
  "filesChanged": [
    "plant-a/pump.assessment.yaml"
  ],
  "assessedCount": 2,
  "diagnostics": []
}
```
**400 — The request body is malformed. Nothing was written.**

```json
{
  "ok": false,
  "reason": "invalid",
  "error": "baseVersion: expected string"
}
```
**403 — The deployment refuses the write, or the file could not be written. Present when the server is configured with a write check.**

```json
{
  "ok": false,
  "reason": "forbidden",
  "error": "This deployment is read-only."
}
```
**404 — No assessment has this id.**

```json
{
  "error": "assessment not found: as-pump"
}
```
**409 — `baseVersion` is not the current version, with `reason` set to `conflict`. Also answers 409 with `reason` set to `ambiguous` when the id names more than one assessment.**

```json
{
  "ok": false,
  "reason": "conflict",
  "current": "c77a...",
  "paths": [
    "plant-a/pump.assessment.yaml"
  ]
}
```
**500 — The run failed before it wrote anything, for example because the bound model could not be loaded. `error` says why.**

```json
{
  "ok": false,
  "reason": "failed",
  "error": "The model could not be loaded."
}
```

## `POST /metron/assessments/{id}/promote`

Creates a finding from outcomes of one assessment, or appends outcomes to an existing finding. With `mode` `create` the server writes the finding into the findings register named by `register` and stamps its `id`, `createdAt`, `createdBy` and `status`. `createdBy` is the assessor recorded in the assessment. With `mode` `append` the server merges `refs` into the finding named by `findingId` and drops duplicates. A request that fails validation answers 400 and writes nothing.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string | The assessment's own id, the `id` that its `.assessment.yaml` file carries. It is not a model id from `/models`. |

### Responses

**200 — The finding was written.**

```json
{
  "ok": true,
  "findingId": "F-A3K9Q6",
  "filesChanged": [
    "findings/network.finding.yaml"
  ]
}
```
**400 — The request is malformed, a register path is not a valid findings register, `refs` is empty or invalid, `label` is blank, or `findingId` names no finding. Nothing was written.**

```json
{
  "ok": false,
  "reason": "invalid",
  "error": "Promote requires at least one ref"
}
```
**403 — The deployment refuses the write, or the file could not be written. Present when the server is configured with a write check.**

```json
{
  "ok": false,
  "reason": "forbidden",
  "error": "This deployment is read-only."
}
```
**404 — No assessment has this id.**

```json
{
  "error": "assessment not found: as-pump"
}
```
**409 — The id names more than one assessment, with `reason` set to `ambiguous`.**

```json
{
  "error": "assessment id as-pump names 2 assessments",
  "reason": "ambiguous",
  "candidates": []
}
```
**500 — The assessment file could not be read. `error` says why.**

```json
{
  "ok": false,
  "reason": "failed",
  "error": "The assessment could not be read."
}
```

## `POST /metron/findings/patches`

Applies an ordered list of edits to existing findings and writes each touched findings register once. An edit sets any of `label`, `description`, `status`, `severity` and `tags`, and an edit with `refs` replaces the finding's refs wholesale. A field an edit leaves out keeps its value. The server checks every edit first. One invalid edit, or one `findingId` that names no finding, answers 400 and writes nothing.

### Responses

**200 — The edits were written.**

```json
{
  "ok": true,
  "filesChanged": [
    "findings/network.finding.yaml"
  ]
}
```
**400 — The request is malformed, an edit is invalid, or a `findingId` names no finding. Nothing was written.**

```json
{
  "ok": false,
  "reason": "invalid",
  "error": "Unknown finding id: F-000000"
}
```
**403 — The deployment refuses the write, or the file could not be written. Present when the server is configured with a write check.**

```json
{
  "ok": false,
  "reason": "forbidden",
  "error": "This deployment is read-only."
}
```

## `GET /metron/packages/{id}/portfolio`

Rolls up every assessment that is pinned to this package and its current version into one tree. An assessment pinned to another version of the package is listed under `excluded` with its reason, and an assessment that fails to load is listed there too. A malformed assessment file is skipped.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string | The package's own id, the `meta.id` that its `.metron.yaml` file carries. It is not a model id from `/models`. |

### Responses

**200 — The roll-up.**

```json
{
  "packageTitle": "Example grid package",
  "packageVersion": 1,
  "tree": {
    "distribution": {
      "yes": 1,
      "no": 0,
      "partial": 0,
      "indeterminate": 0,
      "not-applicable": 0
    },
    "coverage": {
      "draft": 0,
      "inProgress": 0,
      "done": 1,
      "requiresRevalidation": 0,
      "na": 0,
      "total": 1
    },
    "children": []
  },
  "included": [
    {
      "relPath": "plant-a/pump.assessment.yaml",
      "meta": {
        "model": "blockdiagram.tetra.yaml",
        "modelId": "bd-plant-a",
        "packageId": "pkg-grid",
        "packageVersion": 1,
        "assessor": "assessor@example.test",
        "startedAt": "2026-06-04T00:00:00.000Z"
      }
    }
  ],
  "excluded": [
    {
      "relPath": "plant-b/pump.assessment.yaml",
      "meta": {
        "model": "blockdiagram.tetra.yaml",
        "modelId": "bd-plant-b",
        "packageId": "pkg-grid",
        "packageVersion": 2,
        "assessor": "assessor@example.test",
        "startedAt": "2026-06-04T00:00:00.000Z"
      },
      "reason": "pinned to package version 2, current package is v1"
    }
  ]
}
```
**403 — The deployment refuses the read. The body carries the reason. Present only when the server is configured with a read check.**

```json
{
  "error": "This licence does not include Metron data."
}
```
**404 — No package has this id.**

```json
{
  "error": "package not found: pkg-grid"
}
```
**409 — The id names more than one package. `reason` is `ambiguous` and `candidates` lists them.**

```json
{
  "error": "package id pkg-grid names 2 packages",
  "reason": "ambiguous",
  "candidates": [
    {
      "kind": "metron-package",
      "id": "pkg-grid",
      "locator": "example/pkg.metron.yaml"
    }
  ]
}
```
**500 — The roll-up could not be built. `error` says why.**

```json
{
  "error": "The package file could not be read."
}
```

## `GET /metron/packages/{id}/portfolio/export`

Returns the same roll-up as `/metron/packages/{id}/portfolio`, as a file. `format=csv` returns one row for each assessed pair across the included assessments, with a column for the model, its id and the assessor. `format=json` returns the nested roll-up with the list of included and excluded assessments and the time the server generated it.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string | The package's own id, the `meta.id` that its `.metron.yaml` file carries. It is not a model id from `/models`. |
| `format` | query | yes | string | `csv` or `json`. |

### Responses

**200 — The file. `Content-Disposition` names it `<package id>-portfolio.<format>`.**

```csv
model_id,model,assessor,package_id,package_version,requirement_id
bd-plant-a,blockdiagram.tetra.yaml,assessor@example.test,pkg-grid,1,R1

```
**400 — `format` is missing or is not `csv` or `json`.**

```json
{
  "ok": false,
  "reason": "invalid",
  "error": "format must be csv or json"
}
```
**403 — The deployment refuses the read. The body carries the reason. Present only when the server is configured with a read check.**

```json
{
  "error": "This licence does not include Metron data."
}
```
**404 — No package has this id.**

```json
{
  "error": "package not found: pkg-grid"
}
```
**409 — The id names more than one package. `reason` is `ambiguous` and `candidates` lists them.**

```json
{
  "error": "package id pkg-grid names 2 packages",
  "reason": "ambiguous",
  "candidates": [
    {
      "kind": "metron-package",
      "id": "pkg-grid",
      "locator": "example/pkg.metron.yaml"
    }
  ]
}
```
**500 — The roll-up could not be built. `error` says why.**

```json
{
  "error": "The package file could not be read."
}
```
