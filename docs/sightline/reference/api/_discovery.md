## `GET /session`

Answers who the bearer token acts as, so a client can show read-only controls to a reader before it offers an edit. A server without authentication permits every write, so it answers `{ "role": "editor" }` with no user id.

### Responses

**200: The caller's role, and its user id and organisation when the server authenticates.**

```json
{
  "role": "reader",
  "userId": "user-1",
  "orgId": "org-1"
}
```
**401: The request carries no valid bearer token.**

## `GET /discovery/models`

Unlike `/models`, this lists every model kind in the workspace, not tetra models only.

### Responses

**200: The models in the workspace, of every kind.**

```json
{
  "models": [
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
}
```

## `GET /discovery/bindings`

Lists every assessment binding candidate found across the workspace's model directories.

### Responses

**200: Assessment binding candidates.**

```json
{
  "bindings": [
    {
      "modelId": "plant-a",
      "packageId": "rp-1",
      "packageVersion": 1
    }
  ]
}
```

## `GET /discovery/external-data`

Reads every external-data file in the workspace. A file that fails to parse reports its own error alongside the results that did parse.

### Responses

**200: External data results, one entry per source file.**

```json
{
  "files": [
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
}
```

## `GET /discovery/rules`

packageDir must stay inside the workspace root; a pattern that resolves outside the package directory is rejected the same way.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `packageDir` | query | yes | string | A path to a package directory, confined to the workspace root. |
| `pattern` | query | no | string[] | Repeatable. A glob pattern resolved against packageDir. |

### Responses

**200: The rules matched by the given patterns.**

```json
{
  "rules": [
    {
      "rule": {
        "ruleId": "r-1",
        "title": "Cross-zone exposure"
      },
      "sourceFile": "rules/arch.rules.yaml"
    }
  ]
}
```
**400: packageDir is missing, or packageDir or a pattern resolves outside the package directory.**

```json
{
  "error": "packageDir must not escape the workspace root"
}
```

## `GET /discovery/rule-packages`

Lists every rule package installed under the workspace root, one element per id and version, ordered by id then version. A client names a package by `id` and `version`, as a `uses:` entry does. Each element carries the package's `domain`. A package whose domain the caller's licence cannot read is omitted, so the answer is `200` with a shorter list and never `403`.

### Responses

**200: The installed packages.**

```json
{
  "packages": [
    {
      "id": "example-rules",
      "version": 1,
      "name": "Example rules",
      "domain": "tetra"
    },
    {
      "id": "vendor-rules",
      "version": 2,
      "domain": "metron"
    }
  ]
}
```

## `GET /discovery/attribute-sets`

Same parameters and 400 rule as `/discovery/rules`.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `packageDir` | query | yes | string | A path to a package directory, confined to the workspace root. |
| `pattern` | query | no | string[] | Repeatable. A glob pattern resolved against packageDir. |

### Responses

**200: The attribute sets matched by the given patterns.**

```json
{
  "attributeSets": [
    {
      "attributeSet": {
        "id": "s-1",
        "label": "Set 1"
      },
      "sourceFile": "attributes/main.attributes.yaml"
    }
  ]
}
```
**400: packageDir is missing, or packageDir or a pattern resolves outside the package directory.**

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

**200: One outcome per matched file: a parsed scheme, an empty file, or an invalid one.**

```json
{
  "schemes": [
    {
      "sourceFile": "default.scheme.yaml",
      "kind": "scheme"
    }
  ]
}
```
**400: packageDir is missing, or resolves outside the workspace root.**

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

**200: The matched file paths, relative to baseDir.**

```json
{
  "paths": [
    "frags/a.yaml",
    "frags/b.yaml"
  ]
}
```
**400: baseDir is missing, or baseDir or a pattern resolves outside baseDir.**

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

**200: Findings grouped by entity id. The `entities` list is empty when nothing references the model.**

```json
{
  "entities": [
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
}
```
**403: The deployment refuses the read. The body carries the reason. Present only when the server is configured with a read check.**

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

**200: Coverage entries sorted by entity id. The `entities` list is empty when no assessment covers the model.**

```json
{
  "entities": [
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
}
```
**403: The deployment refuses the read. The body carries the reason. Present only when the server is configured with a read check.**

```json
{
  "error": "This licence does not include Metron data in Tetra."
}
```

## `GET /cross-app/models/{modelId}/covering-causes`

Scans every Lamina model in the workspace and returns, for each channel or flow of the Tetra model, the causes whose vector links a rule finding that has that channel or flow among its references. A cause whose link names a different Tetra model is left out. A cause appears once for each channel or flow it covers. This is the read that Tetra uses for its Causes section and cause badges.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The Tetra model's own id, the id that `/models` lists for it. It is not the `meta.modelId` that the findings and coverage reads take. |

### Responses

**200: The covering causes grouped by channel or flow id. The `relations` list is empty when no cause links a rule finding of the model.**

```json
{
  "relations": [
    {
      "relationId": "c-ot-it",
      "causes": [
        {
          "laminaModelId": "example-risk-model",
          "laminaModelName": "Example risk model",
          "causeId": "C-001",
          "causeLabel": "Remote access abused",
          "findingId": "example-rules/cross-zone-relation:example-ot-model/relationId=c-ot-it,zoneId=z-it"
        }
      ]
    }
  ]
}
```
**404: No Tetra model has this id.**

```json
{
  "error": "model not found: example-grid"
}
```
**409: More than one Tetra model has this id.**

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

## `GET /cross-app/models/{modelId}/tetra-scope`

Lists the Tetra models of the workspace, in any folder, each with the zone ids that it declares, sorted by model id. The Lamina model selects and authorises the read, and its own folder does not change the answer. The Model Properties drawer offers them as the choices for `tetraModel` and `tetraZones`. A zone that no relation enters has no vector and is still listed. A Tetra manifest with no `meta.id` is not a listed model and is left out.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The Lamina model's own id (`meta.id` of its `*.bowtie.yaml`). It is not a Tetra model's id. |

### Responses

**200: The Tetra models of the workspace.**

```json
{
  "models": [
    {
      "id": "example-grid",
      "zoneIds": [
        "z-it",
        "z-ot"
      ]
    }
  ]
}
```
**404: No Lamina model has this id.**

```json
{
  "error": "model not found: example-grid"
}
```
**409: More than one Lamina model has this id.**

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

## `GET /cross-app/models/{modelId}/finding-links`

Compares the pin that each cause holds on a record link with the record now, and answers one entry for each record link. The pin (`accepted`) holds the data hash, the source hash and the values the record had. The eight states are `current`, `data-changed`, `source-changed`, `both-changed`, `new`, `closed`, `missing` and `unreadable`. `new` means the link has no accepted pin. `closed` means the record is no longer emitted. `missing` means no record answers to the id. `unreadable` means the register could not be read now. For a changed state, `explanation` names what moved, from the values in the pin or from the journal. A cause with no record link is not listed.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The Lamina model's own id (`meta.id` of its `*.bowtie.yaml`). It is not a Tetra model's id. |

### Responses

**200: One entry for each record link in the model. The `links` list is empty when none exists.**

```json
{
  "links": [
    {
      "causeId": "C-001",
      "findingId": "example-rules/cross-zone-relation:example-grid/relationId=F1,zoneId=Z1",
      "modelId": "example-grid",
      "name": "Relation enters a zone: Z1, F1",
      "state": "data-changed",
      "closed": false,
      "title": "Relation enters a zone: Z1, F1",
      "dataHash": "1a2b3c4d5e6f7",
      "sourceHash": "7f6e5d4c3b2a1",
      "linkIds": [
        "F1"
      ],
      "explanation": {
        "data": [
          {
            "path": "type",
            "old": "system",
            "new": "security"
          }
        ]
      }
    }
  ]
}
```
**403: The deployment refuses the read. The body carries the reason. Present only when the server is configured with a read check.**

```json
{
  "error": "This licence does not include Metron data in Tetra."
}
```
**404: No Lamina model has this id.**

```json
{
  "error": "model not found: example-grid"
}
```
**409: More than one Lamina model has this id.**

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
**500: The model exists but failed to load.**

```json
{
  "error": "unable to load example-grid"
}
```

## `GET /cross-app/models/{modelId}/causes/{causeId}/finding-links`

Answers the same entries as the model-wide route, for one cause. The `links` list is empty when the cause links no record.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The Lamina model's own id (`meta.id` of its `*.bowtie.yaml`). It is not a Tetra model's id. |
| `causeId` | path | yes | string | The `id` of a cause in the Lamina model. |

### Responses

**200: The record links of the cause.**

```json
{
  "links": [
    {
      "causeId": "C-001",
      "findingId": "example-rules/cross-zone-relation:example-grid/relationId=F1,zoneId=Z1",
      "modelId": "example-grid",
      "name": "Relation enters a zone: Z1, F1",
      "state": "data-changed",
      "closed": false,
      "title": "Relation enters a zone: Z1, F1",
      "dataHash": "1a2b3c4d5e6f7",
      "sourceHash": "7f6e5d4c3b2a1",
      "linkIds": [
        "F1"
      ],
      "explanation": {
        "data": [
          {
            "path": "type",
            "old": "system",
            "new": "security"
          }
        ]
      }
    }
  ]
}
```
**403: The deployment refuses the read. The body carries the reason. Present only when the server is configured with a read check.**

```json
{
  "error": "This licence does not include Metron data in Tetra."
}
```
**404: No Lamina model has this id, or the Lamina model has no cause with this id.**

```json
{
  "error": "cause not found: C-001"
}
```
**409: More than one Lamina model has this id.**

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
**500: The model exists but failed to load.**

```json
{
  "error": "unable to load example-grid"
}
```

## `GET /cross-app/models/{modelId}/risks`

A Risk groups cause-event-outcome paths. The manifest's `risks` list defines each Risk by the elements it includes, the elements it excludes, an optional program and the Risks it holds. A Risk counts only its members: a path is a member event with one of its member causes and one of its member outcomes. A model whose manifest defines no Risks answers no Risks. A path can sit in several Risks. Each Risk with paths carries a residual and an inherent `rating`, and `ratingBasis` names the risk matrix and the likelihood rule that produced it. A Risk's own `aggregation` and the manifest's `riskDefaults` set how its causes and events combine, and the default is `simple-damped-union`. `issues` lists the definitions that the read could not fully honour, and `problems` lists the files that could not be read.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The Lamina model's own id (`meta.id` of its `*.bowtie.yaml`). It is not a Tetra model's id. |

### Responses

**200: The Risks of the model.**

```json
{
  "risks": [
    {
      "id": "R001",
      "name": "Station outage",
      "source": "authored",
      "eventIds": [
        "E-001",
        "E-002"
      ],
      "paths": [
        {
          "causeId": "C-001",
          "eventId": "E-001",
          "outcomeId": "O-001"
        },
        {
          "causeId": "C-002",
          "eventId": "E-001",
          "outcomeId": "O-001"
        },
        {
          "causeId": "C-001",
          "eventId": "E-002",
          "outcomeId": "O-002"
        }
      ]
    },
    {
      "id": "R002",
      "name": "Remote cause, one outcome",
      "source": "authored",
      "eventIds": [
        "E-001"
      ],
      "paths": [
        {
          "causeId": "C-002",
          "eventId": "E-001",
          "outcomeId": "O-002"
        }
      ]
    }
  ],
  "membership": {
    "byElement": {
      "C-001": [
        "R001"
      ]
    },
    "byRisk": {}
  },
  "uncoveredEventIds": [
    "E-003"
  ],
  "issues": [
    {
      "riskId": "R003",
      "kind": "unknown-element",
      "id": "E-009"
    }
  ],
  "problems": []
}
```
**404: No Lamina model has this id.**

```json
{
  "error": "model not found: example-grid"
}
```
**409: More than one Lamina model has this id.**

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
**500: The model exists but failed to load.**

```json
{
  "error": "unable to load example-grid"
}
```

## `GET /cross-app/models/{modelId}/control-templates/{templateId}/impact`

Collects every control in the model that names the template, and answers the Risks those controls reach. An instance is a `gap` when its current effectiveness rating is below its target on None, Low, Partial and Full, and `unassessed` when either rating is missing or Not Assessed; both make each Risk they reach a gap risk. Not Applicable on either rating is never a gap. A Risk is critical when a control that reaches it has `critical: true`. Ratings and the critical flag are read after the control's template is applied, so a control that omits them carries the template's values. A control reaches a Risk when its owner is a member of the Risk, or of a Risk that it holds: the cause for a preventative control and the outcome for a mitigative one. A model that defines no Risks answers no Risks. Each Risk carries its residual and inherent `rating`, and `ratingBasis` names the risk matrix and the likelihood rule the ratings used. `issues` lists the Risk definitions that the read could not fully honour. `problems` lists the files of the model that could not be read or parsed, and an answer with problems may understate the impact. A template the model defines but no control names answers an empty result. A control is reused by template, so this is the way to ask about "the control" across causes and outcomes.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The Lamina model's own id (`meta.id` of its `*.bowtie.yaml`). It is not a Tetra model's id. |
| `templateId` | path | yes | string | The id of a control template (`tpl-control-…`), as a control's `template` field names it. |
| `zone` | query | no | string | Keeps the instances whose zones include this zone id. An instance takes its own `zones`, else its owner's. The answer is empty, not an error, when none match. |

### Responses

**200: The matched controls, the events they reach, and a summary.**

```json
{
  "control": {
    "templateId": "tpl-access-local",
    "label": "Local access control",
    "controlType": "Preventative",
    "category": "Technology",
    "group": "IAM"
  },
  "instances": [
    {
      "controlId": "K-001",
      "ownerKind": "cause",
      "ownerId": "C-001",
      "templateId": "tpl-access-local",
      "zones": [
        "Z-STATION"
      ],
      "targetEffRating": "Partial",
      "currentEffRating": "None",
      "critical": true,
      "state": "gap",
      "currentEffDetail": "Field devices have default or no credentials.",
      "eventIds": [
        "E-001",
        "E-002"
      ]
    },
    {
      "controlId": "K-002",
      "ownerKind": "cause",
      "ownerId": "C-002",
      "templateId": "tpl-access-local",
      "zones": [
        "Z-STATION"
      ],
      "targetEffRating": "Partial",
      "currentEffRating": "Full",
      "critical": true,
      "state": "met",
      "eventIds": [
        "E-002"
      ]
    }
  ],
  "risks": [
    {
      "eventId": "E-001",
      "label": "Breakers opened",
      "causes": [
        {
          "id": "C-001",
          "label": "Physical access",
          "inherentLikelihood": 2,
          "currentLikelihood": 2
        }
      ],
      "outcomes": [
        {
          "id": "O-001",
          "label": "Station blackout"
        }
      ],
      "instanceIds": [
        "K-001"
      ],
      "gap": true,
      "critical": true
    },
    {
      "eventId": "E-002",
      "label": "Configurations wiped",
      "causes": [
        {
          "id": "C-001"
        },
        {
          "id": "C-002"
        }
      ],
      "outcomes": [
        {
          "id": "O-002",
          "label": "Equipment damage"
        }
      ],
      "instanceIds": [
        "K-001",
        "K-002"
      ],
      "gap": true,
      "critical": true
    }
  ],
  "summary": {
    "instanceCount": 2,
    "riskCount": 2,
    "gapRiskIds": [
      "E-001",
      "E-002"
    ],
    "criticalRiskIds": [
      "E-001",
      "E-002"
    ],
    "byState": {
      "gap": 1,
      "met": 1,
      "unassessed": 0,
      "not-applicable": 0
    }
  },
  "problems": []
}
```
**404: No Lamina model has this id, or the model neither defines nor uses the template. The `error` text names which.**

```json
{
  "error": "template not found: tpl-access-local"
}
```
**409: More than one Lamina model has this id.**

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
**500: The model exists but failed to load.**

```json
{
  "error": "unable to load example-grid"
}
```

## `GET /cross-app/models/{modelId}/causes/{causeId}/controls/{controlId}/impact`

Answers for the control `controlId` that the cause `causeId` owns, which is a preventative control. An instance is a `gap` when its current effectiveness rating is below its target on None, Low, Partial and Full, and `unassessed` when either rating is missing or Not Assessed; both make each Risk they reach a gap risk. Not Applicable on either rating is never a gap. A Risk is critical when a control that reaches it has `critical: true`. Ratings and the critical flag are read after the control's template is applied, so a control that omits them carries the template's values. A control reaches a Risk when its owner is a member of the Risk, or of a Risk that it holds: the cause for a preventative control and the outcome for a mitigative one. A model that defines no Risks answers no Risks. Each Risk carries its residual and inherent `rating`, and `ratingBasis` names the risk matrix and the likelihood rule the ratings used. `issues` lists the Risk definitions that the read could not fully honour. `problems` lists the files of the model that could not be read or parsed, and an answer with problems may understate the impact. With `scope=template` the answer widens to every control that names the same template.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The Lamina model's own id (`meta.id` of its `*.bowtie.yaml`). It is not a Tetra model's id. |
| `causeId` | path | yes | string | The `id` of a cause in the Lamina model. |
| `controlId` | path | yes | string | The `id` of a control the cause owns. |
| `scope` | query | no | string | `instance` (the default) answers the one control. `template` answers every control that names the same template as this one; a control that names no template answers itself alone. |
| `zone` | query | no | string | Keeps the instances whose zones include this zone id. An instance takes its own `zones`, else its owner's. The answer is empty, not an error, when none match. |

### Responses

**200: The matched controls, the events they reach, and a summary.**

```json
{
  "control": {
    "templateId": "tpl-access-local",
    "label": "Local access control",
    "controlType": "Preventative",
    "category": "Technology",
    "group": "IAM"
  },
  "instances": [
    {
      "controlId": "K-001",
      "ownerKind": "cause",
      "ownerId": "C-001",
      "templateId": "tpl-access-local",
      "zones": [
        "Z-STATION"
      ],
      "targetEffRating": "Partial",
      "currentEffRating": "None",
      "critical": true,
      "state": "gap",
      "currentEffDetail": "Field devices have default or no credentials.",
      "eventIds": [
        "E-001",
        "E-002"
      ]
    },
    {
      "controlId": "K-002",
      "ownerKind": "cause",
      "ownerId": "C-002",
      "templateId": "tpl-access-local",
      "zones": [
        "Z-STATION"
      ],
      "targetEffRating": "Partial",
      "currentEffRating": "Full",
      "critical": true,
      "state": "met",
      "eventIds": [
        "E-002"
      ]
    }
  ],
  "risks": [
    {
      "eventId": "E-001",
      "label": "Breakers opened",
      "causes": [
        {
          "id": "C-001",
          "label": "Physical access",
          "inherentLikelihood": 2,
          "currentLikelihood": 2
        }
      ],
      "outcomes": [
        {
          "id": "O-001",
          "label": "Station blackout"
        }
      ],
      "instanceIds": [
        "K-001"
      ],
      "gap": true,
      "critical": true
    },
    {
      "eventId": "E-002",
      "label": "Configurations wiped",
      "causes": [
        {
          "id": "C-001"
        },
        {
          "id": "C-002"
        }
      ],
      "outcomes": [
        {
          "id": "O-002",
          "label": "Equipment damage"
        }
      ],
      "instanceIds": [
        "K-001",
        "K-002"
      ],
      "gap": true,
      "critical": true
    }
  ],
  "summary": {
    "instanceCount": 2,
    "riskCount": 2,
    "gapRiskIds": [
      "E-001",
      "E-002"
    ],
    "criticalRiskIds": [
      "E-001",
      "E-002"
    ],
    "byState": {
      "gap": 1,
      "met": 1,
      "unassessed": 0,
      "not-applicable": 0
    }
  },
  "problems": []
}
```
**400: `scope` is neither `instance` nor `template`.**

```json
{
  "error": "scope must be \"instance\" or \"template\": all"
}
```
**404: No Lamina model has this id, the model has no such cause, or the cause owns no such control. A control of another owner does not match. The `error` text names which.**

```json
{
  "error": "control not found: K-001"
}
```
**409: More than one Lamina model has this id.**

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
**500: The model exists but failed to load.**

```json
{
  "error": "unable to load example-grid"
}
```

## `GET /cross-app/models/{modelId}/outcomes/{outcomeId}/controls/{controlId}/impact`

Answers for the control `controlId` that the outcome `outcomeId` owns, which is a mitigative control. An instance is a `gap` when its current effectiveness rating is below its target on None, Low, Partial and Full, and `unassessed` when either rating is missing or Not Assessed; both make each Risk they reach a gap risk. Not Applicable on either rating is never a gap. A Risk is critical when a control that reaches it has `critical: true`. Ratings and the critical flag are read after the control's template is applied, so a control that omits them carries the template's values. A control reaches a Risk when its owner is a member of the Risk, or of a Risk that it holds: the cause for a preventative control and the outcome for a mitigative one. A model that defines no Risks answers no Risks. Each Risk carries its residual and inherent `rating`, and `ratingBasis` names the risk matrix and the likelihood rule the ratings used. `issues` lists the Risk definitions that the read could not fully honour. `problems` lists the files of the model that could not be read or parsed, and an answer with problems may understate the impact. With `scope=template` the answer widens to every control that names the same template.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The Lamina model's own id (`meta.id` of its `*.bowtie.yaml`). It is not a Tetra model's id. |
| `outcomeId` | path | yes | string | The `id` of an outcome in the Lamina model. |
| `controlId` | path | yes | string | The `id` of a control the outcome owns. |
| `scope` | query | no | string | `instance` (the default) answers the one control. `template` answers every control that names the same template as this one; a control that names no template answers itself alone. |
| `zone` | query | no | string | Keeps the instances whose zones include this zone id. An instance takes its own `zones`, else its owner's. The answer is empty, not an error, when none match. |

### Responses

**200: The matched controls, the events they reach, and a summary.**

```json
{
  "control": {
    "templateId": "tpl-access-local",
    "label": "Local access control",
    "controlType": "Preventative",
    "category": "Technology",
    "group": "IAM"
  },
  "instances": [
    {
      "controlId": "K-001",
      "ownerKind": "cause",
      "ownerId": "C-001",
      "templateId": "tpl-access-local",
      "zones": [
        "Z-STATION"
      ],
      "targetEffRating": "Partial",
      "currentEffRating": "None",
      "critical": true,
      "state": "gap",
      "currentEffDetail": "Field devices have default or no credentials.",
      "eventIds": [
        "E-001",
        "E-002"
      ]
    },
    {
      "controlId": "K-002",
      "ownerKind": "cause",
      "ownerId": "C-002",
      "templateId": "tpl-access-local",
      "zones": [
        "Z-STATION"
      ],
      "targetEffRating": "Partial",
      "currentEffRating": "Full",
      "critical": true,
      "state": "met",
      "eventIds": [
        "E-002"
      ]
    }
  ],
  "risks": [
    {
      "eventId": "E-001",
      "label": "Breakers opened",
      "causes": [
        {
          "id": "C-001",
          "label": "Physical access",
          "inherentLikelihood": 2,
          "currentLikelihood": 2
        }
      ],
      "outcomes": [
        {
          "id": "O-001",
          "label": "Station blackout"
        }
      ],
      "instanceIds": [
        "K-001"
      ],
      "gap": true,
      "critical": true
    },
    {
      "eventId": "E-002",
      "label": "Configurations wiped",
      "causes": [
        {
          "id": "C-001"
        },
        {
          "id": "C-002"
        }
      ],
      "outcomes": [
        {
          "id": "O-002",
          "label": "Equipment damage"
        }
      ],
      "instanceIds": [
        "K-001",
        "K-002"
      ],
      "gap": true,
      "critical": true
    }
  ],
  "summary": {
    "instanceCount": 2,
    "riskCount": 2,
    "gapRiskIds": [
      "E-001",
      "E-002"
    ],
    "criticalRiskIds": [
      "E-001",
      "E-002"
    ],
    "byState": {
      "gap": 1,
      "met": 1,
      "unassessed": 0,
      "not-applicable": 0
    }
  },
  "problems": []
}
```
**400: `scope` is neither `instance` nor `template`.**

```json
{
  "error": "scope must be \"instance\" or \"template\": all"
}
```
**404: No Lamina model has this id, the model has no such outcome, or the outcome owns no such control. A control of another owner does not match. The `error` text names which.**

```json
{
  "error": "outcome not found: O-001"
}
```
**409: More than one Lamina model has this id.**

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
**500: The model exists but failed to load.**

```json
{
  "error": "unable to load example-grid"
}
```

## `POST /cross-app/models/{modelId}/risks/query`

Rates one Risk without defining it in the manifest. Send `riskId` to rate a Risk the model defines. Send `include` to rate a Risk that you build: `include` lists the cause, event and outcome ids the Risk counts, `exclude` lists ids it leaves out, `filter.program` is an optional program that runs first, and `risks` lists Risks the manifest defines that it holds. Exclude beats include, and both beat the program. `riskId` and the built form exclude each other; `id` (default `adhoc`), `name`, `aggregation` and `flattenSubRisks` are optional, and `aggregation` overrides the manifest's `riskDefaults` for this query only. The answer carries `kind`, `memberIds` and `missingNeighbourIds`, as the list route gives them. A held Risk is a Risk the manifest defines. The route takes a body, so it is a `POST`, but it only reads: it writes nothing, takes no lock and is not audited, and a read-only caller may use it. The answer is the Risk as the list route gives it, with the same `ratingBasis`. `issues` lists only the definition issues of the queried Risk, and `problems` lists the files that could not be read. An `include`, `exclude` or `risks` entry that names an unknown element or Risk adds an issue and does not fail the request.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The Lamina model's own id (`meta.id` of its `*.bowtie.yaml`). It is not a Tetra model's id. |

### Responses

**200: The rated Risk.**

```json
{
  "risk": {
    "id": "station-outage",
    "name": "Station outage",
    "source": "authored",
    "eventIds": [
      "E-001",
      "E-002"
    ],
    "subRiskIds": [],
    "paths": [
      {
        "causeId": "C-001",
        "eventId": "E-001",
        "outcomeId": "O-001"
      },
      {
        "causeId": "C-001",
        "eventId": "E-002",
        "outcomeId": "O-002"
      }
    ]
  },
  "ratingBasis": {
    "riskMatrixDef": "default",
    "aggregationMode": "simple-damped-union"
  },
  "issues": [],
  "problems": []
}
```
**400: The body is not JSON, is not a Risk query, or its `id` is one the manifest already defines. The last answer carries `reason: "duplicate-id"`.**

```json
{
  "error": "the manifest already defines a Risk with the id R001",
  "reason": "duplicate-id"
}
```
**404: No Lamina model has this id, or the model has no Risk with `riskId`.**

```json
{
  "error": "risk not found: R009"
}
```
**409: More than one Lamina model has this id.**

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
**500: The model exists but failed to load.**

```json
{
  "error": "unable to load example-grid"
}
```

## `GET /cross-app/models/{modelId}/finding-inbox`

Lists the open records that their rule does not hide. The scope is the Tetra model that the Lamina manifest names under `tetraModel`, or every Tetra model the workspace lists when it names none. `findings` holds the Tetra records in the zones the manifest declares under `tetraZones`. `metron` holds the Metron findings of the same model scope, which need no zones. Each entry is marked `linked` when some cause of this Lamina model links it, so the answer to unlinked is per Lamina model.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The Lamina model's own id (`meta.id` of its `*.bowtie.yaml`). It is not a Tetra model's id. |

### Responses

**200: The records in scope. `findings` is empty when `state` is not `ready`, and `metron` is empty only on `model-not-found`.**

```json
{
  "state": "ready",
  "findings": [
    {
      "id": "example-rules/cross-zone-relation:example-grid/relationId=F1,zoneId=Z1",
      "packageId": "example-rules",
      "modelId": "example-grid",
      "ruleId": "cross-zone-relation",
      "title": "Relation enters a zone: Z1, F1",
      "key": {
        "relationId": "F1",
        "zoneId": "Z1"
      },
      "zoneId": "Z1",
      "linked": false,
      "state": "current",
      "dataHash": "1a2b3c4d5e6f7",
      "sourceHash": "7f6e5d4c3b2a1"
    }
  ],
  "metron": [
    {
      "id": "assessment-findings/assessment-gap:example-grid/entityId=z-1,legacyId=F-1,requirementId=R-1",
      "packageId": "assessment-findings",
      "modelId": "example-grid",
      "ruleId": "assessment-gap",
      "title": "Backups are untested",
      "description": "No restore drill has been run.",
      "severity": "high",
      "key": {
        "entityId": "z-1",
        "legacyId": "F-1",
        "requirementId": "R-1"
      },
      "linked": false,
      "state": "current",
      "dataHash": "1a2b3c4d5e6f7",
      "sourceHash": "7f6e5d4c3b2a1"
    }
  ]
}
```
**403: The deployment refuses the read. The body carries the reason. Present only when the server is configured with a read check.**

```json
{
  "error": "This licence does not include Metron data in Tetra."
}
```
**404: No Lamina model has this id.**

```json
{
  "error": "model not found: example-grid"
}
```
**409: More than one Lamina model has this id.**

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
**500: The model exists but failed to load.**

```json
{
  "error": "unable to load example-grid"
}
```

## `GET /metron/assessments/{id}`

Returns the parsed assessment and a version. The version is a hash of the assessment file's text. Send it back as `baseVersion` when you post edits to `/metron/assessments/{id}/ops`.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string | The assessment's own id, the `id` that its `.assessment.yaml` file carries. It is not a model id from `/models`. |

### Responses

**200: The assessment and its version.**

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
**403: The deployment refuses the read. The body carries the reason. Present only when the server is configured with a read check.**

```json
{
  "error": "This licence does not include Metron data."
}
```
**404: No assessment has this id.**

```json
{
  "error": "assessment not found: as-pump"
}
```
**409: The id names more than one assessment. `reason` is `ambiguous` and `candidates` lists them.**

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

## `GET /metron/assessments/{id}/workspace`

Returns the text of every file the assessment's window reads, so a browser that has no file system can open it. The set is what the assessment's loaders read: the assessment, the model it is bound to, its package, the assessment's own register and the external-data tables. Paths are relative to the model root and no absolute path appears. A file that cannot be read is left out and the window reports the error itself.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string | The assessment's own id, the `id` that its `.assessment.yaml` file carries. It is not a model id from `/models`. |

### Responses

**200: The assessment's version, its entry and the files its window reads.**

```json
{
  "version": "4be1...",
  "entry": "plant-a/pump.assessment.yaml",
  "modelEntry": "plant-a/blockdiagram.tetra.yaml",
  "files": {
    "plant-a/pump.assessment.yaml": "id: as-pump\n...",
    "plant-a/blockdiagram.tetra.yaml": "id: bd-plant-a\n..."
  }
}
```
**403: The deployment refuses the read. The body carries the reason. Present only when the server is configured with a read check.**

```json
{
  "error": "This licence does not include Metron data."
}
```
**404: No assessment has this id.**

```json
{
  "error": "assessment not found: as-pump"
}
```

## `POST /metron/assessments/{id}/ops`

Applies the edits in `ops` in order and writes the assessment file once. The same commit also writes the assessment's own register: each pair's outcome, state and evidence is set as the `person` layer of its record, a pair's shared assessment is a manual link on its record, the versions a done pair was assessed at are the `accepted` pins of its record's links, and a record is created for a pair that has none. The shared assessments and the general evidence are records of the same register. A damaged register answers 400 and nothing is written. Answers 400 invalid when the assessment file still holds `pairs`, `sharedAssessments` or `generalEvidence`, or when the register still holds a record in an earlier shape; run the backfill first. The batch applies only when `baseVersion` matches the current version, so a stale write answers 409 with the current version instead of overwriting a newer edit. If any edit fails, nothing is written. The server stamps `by` and `at` on every log entry it writes, from the assessment's assessor and its own clock. A request cannot set them. The edit kinds are `applyAssessment`, `setPairState`, `acceptPairs`, `reconcileBinding`, `archiveOrphan`, `setGeneralEvidence`, `setSharedAssessments`, `saveSlice` and `deleteSlice`.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string | The assessment's own id, the `id` that its `.assessment.yaml` file carries. It is not a model id from `/models`. |

### Responses

**200: The batch applied.**

```json
{
  "ok": true,
  "version": "c77a...",
  "filesChanged": [
    "plant-a/pump.assessment.yaml"
  ]
}
```
**400: The body is malformed, an edit is invalid, or an edit failed. Nothing was written. `error` says which.**

```json
{
  "ok": false,
  "reason": "invalid",
  "error": "metronArchiveOrphan: no pair R9/E9 to archive"
}
```
**403: The deployment refuses the write, or the file could not be written. Present when the server is configured with a write check.**

```json
{
  "ok": false,
  "reason": "forbidden",
  "error": "This deployment is read-only."
}
```
**404: No assessment has this id.**

```json
{
  "error": "assessment not found: as-pump"
}
```
**409: `baseVersion` is not the current version, with `reason` set to `conflict`. Also answers 409 with `reason` set to `ambiguous` when the id names more than one assessment.**

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

Assesses every pair of a rule-assessed requirement that has no outcome yet, and writes the assessment file once. Each outcome lands as a draft, and a rule never completes a pair. Pairs that already hold an outcome are never changed. The run also keeps the assessment's records, one for each requirement and in-scope entity, in the assessment's own register, and writes them in the same commit as the assessment file. Each record also carries its pair's outcome, state and evidence as its `person` layer. The request body is optional. When it carries `baseVersion`, the run applies only if that matches the current version, and a stale run answers 409 without writing. A run that changes nothing answers 200 with `filesChanged` empty and the current version. Answers 400 invalid when the assessment file still holds `pairs`, `sharedAssessments` or `generalEvidence`, or when the register still holds a record in an earlier shape; run the backfill first.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string | The assessment's own id, the `id` that its `.assessment.yaml` file carries. It is not a model id from `/models`. |

### Responses

**200: The run finished.**

```json
{
  "ok": true,
  "version": "9c2d...",
  "filesChanged": [
    "plant-a/pump.assessment.yaml"
  ],
  "assessedCount": 2,
  "diagnostics": [],
  "records": {
    "created": 2,
    "changed": 0,
    "closed": 0,
    "returned": 0
  }
}
```
**400: The request body is malformed. Nothing was written.**

```json
{
  "ok": false,
  "reason": "invalid",
  "error": "baseVersion: expected string"
}
```
**403: The deployment refuses the write, or the file could not be written. Present when the server is configured with a write check.**

```json
{
  "ok": false,
  "reason": "forbidden",
  "error": "This deployment is read-only."
}
```
**404: No assessment has this id.**

```json
{
  "error": "assessment not found: as-pump"
}
```
**409: `baseVersion` is not the current version, with `reason` set to `conflict`. Also answers 409 with `reason` set to `ambiguous` when the id names more than one assessment.**

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
**500: The run failed before it wrote anything, for example because the bound model could not be loaded. `error` says why.**

```json
{
  "ok": false,
  "reason": "failed",
  "error": "The model could not be loaded."
}
```

## `POST /metron/packages/{id}/assessments`

Applies one requirement package to one Tetra model. The server writes an empty assessment file beside the model, named for the model, the package and its version, and answers the new assessment's id. When that assessment already exists, the server writes nothing and answers its id with `created` false. The assessor is the authenticated user, or `anonymous` when the server has no authentication.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string | The package's own id, the `meta.id` its manifest carries. |
| `version` | query | no | integer | The package version. Required when the package exists in several versions. |

### Responses

**200: An assessment of this package version and model already exists. Nothing was written.**

```json
{
  "ok": true,
  "id": "as-pump",
  "locator": "plant-a/blockdiagram.pkg-grid-v2.assessment.yaml",
  "created": false,
  "filesChanged": []
}
```
**201: The assessment was created.**

```json
{
  "ok": true,
  "id": "as-pump",
  "locator": "plant-a/blockdiagram.pkg-grid-v2.assessment.yaml",
  "created": true,
  "filesChanged": [
    "plant-a/blockdiagram.pkg-grid-v2.assessment.yaml"
  ]
}
```
**400: The body or the `version` query is malformed, or the package has no version. Nothing was written.**

```json
{
  "ok": false,
  "reason": "invalid",
  "error": "version must be an integer"
}
```
**403: The deployment refuses the write, or the file could not be written. Present when the server is configured with a write check.**

```json
{
  "ok": false,
  "reason": "forbidden",
  "error": "This deployment is read-only."
}
```
**404: No requirement package or no Tetra model has this id.**

```json
{
  "error": "model not found: bd-plant-a"
}
```
**409: The id names several packages or models, with `reason` set to `ambiguous` and the `candidates`.**

```json
{
  "error": "model id bd-plant-a names 2 models",
  "reason": "ambiguous",
  "candidates": [
    {
      "kind": "tetra",
      "id": "bd-plant-a",
      "locator": "plant-a/blockdiagram.tetra.yaml"
    },
    {
      "kind": "tetra",
      "id": "bd-plant-a",
      "locator": "plant-b/other.tetra.yaml"
    }
  ]
}
```

## `POST /metron/packages/{id}/versions`

Scaffolds version N+1 of one requirement package as a sibling directory. The server copies the files the package loads, rewrites the manifest with the next `meta.version` and a matching `-vN` in the directory, file and `meta.name`, and answers the files it wrote. It never copies or changes an assessment or a finding. The source package is not changed. The request has no body.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string | The package's own id, the `meta.id` its manifest carries. |
| `version` | query | no | integer | The version to copy from. Required when the package exists in several versions. |

### Responses

**201: The new version was written.**

```json
{
  "ok": true,
  "id": "pkg-grid",
  "fromVersion": 1,
  "toVersion": 2,
  "locator": "pkg-grid-v2/pkg-grid-v2.metron.yaml",
  "filesChanged": [
    "pkg-grid-v2/pkg-grid-v2.metron.yaml",
    "pkg-grid-v2/requirements/general.req.yaml"
  ],
  "warnings": [],
  "notes": []
}
```
**400: The `version` query is not an integer. Nothing was written.**

```json
{
  "ok": false,
  "reason": "invalid",
  "error": "version must be an integer"
}
```
**403: The deployment refuses the write. Present when the server is configured with a write check.**

```json
{
  "ok": false,
  "reason": "forbidden",
  "error": "This deployment is read-only."
}
```
**404: No requirement package has this id.**

```json
{
  "error": "package not found: pkg-grid"
}
```
**409: The package exists in several versions and no `version` was given (`reason` is `ambiguous`), or the new version cannot be created (`reason` is `refused`): its directory already exists and holds files, the package folder holds a nested component package, or another manifest already has the new identity. `error` says which. Nothing was written.**

```json
{
  "ok": false,
  "reason": "refused",
  "error": "pkg-grid-v2 already exists and is not empty. Move it or delete it, then re-run."
}
```
**500: The new version could not be written completely. Delete any partial directory beside the package, then try again.**

```json
{
  "ok": false,
  "reason": "write-failed",
  "error": "The new version could not be written. Delete any partial directory beside the package, then try again."
}
```

## `GET /metron/packages/{id}/portfolio`

Rolls up every assessment that is pinned to this package and its current version into one tree. An assessment pinned to another version of the package is listed under `excluded` with its reason, and an assessment that fails to load is listed there too. A malformed assessment file is skipped.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string | The package's own id, the `meta.id` that its `.metron.yaml` file carries. It is not a model id from `/models`. |
| `version` | query | no | integer | The package version to roll up. A package can exist in several versions under one id, and without this parameter the route answers 409 for such a package. |

### Responses

**200: The roll-up.**

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
**400: `version` is not an integer.**

```json
{
  "ok": false,
  "reason": "invalid",
  "error": "version must be an integer"
}
```
**403: The deployment refuses the read. The body carries the reason. Present only when the server is configured with a read check.**

```json
{
  "error": "This licence does not include Metron data."
}
```
**404: No package has this id.**

```json
{
  "error": "package not found: pkg-grid"
}
```
**409: The id names more than one package. `reason` is `ambiguous` and `candidates` lists them.**

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
**500: The roll-up could not be built. `error` says why.**

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
| `version` | query | no | integer | The package version to roll up. A package can exist in several versions under one id, and without this parameter the route answers 409 for such a package. |

### Responses

**200: The file. `Content-Disposition` names it `<package id>-portfolio.<format>`.**

```csv
model_id,model,assessor,package_id,package_version,requirement_id
bd-plant-a,blockdiagram.tetra.yaml,assessor@example.test,pkg-grid,1,R1

```
**400: `format` is missing or is not `csv` or `json`, or `version` is not an integer.**

```json
{
  "ok": false,
  "reason": "invalid",
  "error": "format must be csv or json"
}
```
**403: The deployment refuses the read. The body carries the reason. Present only when the server is configured with a read check.**

```json
{
  "error": "This licence does not include Metron data."
}
```
**404: No package has this id.**

```json
{
  "error": "package not found: pkg-grid"
}
```
**409: The id names more than one package. `reason` is `ambiguous` and `candidates` lists them.**

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
**500: The roll-up could not be built. `error` says why.**

```json
{
  "error": "The package file could not be read."
}
```

## `GET /folders`

Lists the workspace root and its subfolders, with siblings sorted, as paths relative to the root (`.` is the root). It skips folders that start with a dot and `node_modules`. `truncated` is true when the list stopped at its cap. Use a value as `location` on `POST /models`, or as `parent` on `POST /folders`.

### Responses

**200: The folders.**

```json
{
  "folders": [
    ".",
    "sites",
    "sites/north"
  ],
  "truncated": false
}
```
**403: The read was refused.**

```json
{
  "error": "Reading Metron data is not permitted."
}
```

## `POST /folders`

Creates one folder under `parent`, a folder that `GET /folders` lists, so that a model or package can be created in it with `POST /models`. The server must be configured to create models. `name` is one path segment: it has no separator, does not start with a dot and is not `node_modules`. A folder that already exists answers 409 with `reason` set to `exists` and is left as it was.

### Responses

**201: The folder was created.**

```json
{
  "folder": "sites/north"
}
```
**400: The body is not JSON. `name` is missing or is not one path segment. `parent` is not a string, leaves the workspace or is not an existing folder.**

```json
{
  "error": "name must not contain a path separator or control character"
}
```
**403: The caller is authenticated as a reader. This is the answer only when the server requires authentication.**

```json
{
  "error": "the editor role is required to write"
}
```
**409: A folder with that name already exists under `parent`. Nothing is changed.**

```json
{
  "ok": false,
  "reason": "exists"
}
```
**500: The folder could not be created.**

```json
{
  "error": "the folder could not be created"
}
```

## `GET /edit-grants`

Reports, for the caller's token, whether saving is allowed in each app (`tetra`, `lamina`, `metron`) and why it is refused when it is not. A page uses it to disable Save before a write fails. The answer is affordance only: a write route still refuses what a grant would refuse. The route exists only on a server that gates editing, and answers 404 on one that does not.

### Responses

**200: One grant for each app.**

```json
{
  "grants": [
    {
      "app": "tetra",
      "allowed": true
    },
    {
      "app": "lamina",
      "allowed": false,
      "reason": "This licence does not include editing in Lamina."
    },
    {
      "app": "metron",
      "allowed": true
    }
  ]
}
```

## `GET /findings`

Lists the records of every register in the workspace, each with its derived state. A record is Current, Data changed, Source changed or Both changed, relative to the values last accepted, and Open or Closed, relative to the latest run. Filters combine with AND. An empty result is `{ "records": [] }`. With no `domain` filter the list omits the records of a domain the deployment refuses, and answers 403 only when it refuses every domain present. A `domain` filter that the deployment refuses answers 403.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | query | no | string | Only the records of this model. |
| `packageId` | query | no | string | Only the records of this rule package. |
| `ruleId` | query | no | string | Only the records of this rule. |
| `domain` | query | no | string | Only the records of this domain, such as `tetra`. |
| `hide` | query | no | boolean | `true` for records whose rule hides them from consumer lists, `false` for the rest. |
| `key.<part>` | query | no | string | Only records whose key has this part with this value, for example `key.zoneId=Z1`. Repeat for several parts. |
| `links.id` | query | no | string | Only records that list a reference to this entity id. |

### Responses

**200: The matching records.**

```json
{
  "records": [
    {
      "id": "example-rules/cross-zone-relation:example-grid/relationId=F1,zoneId=Z1",
      "packageId": "example-rules",
      "modelId": "example-grid",
      "hide": true,
      "record": {
        "source": {
          "ruleId": "cross-zone-relation",
          "key": {
            "zoneId": "Z1",
            "relationId": "F1"
          }
        },
        "domain": "tetra",
        "data": {
          "channelType": "network"
        },
        "links": [
          {
            "id": "F1",
            "version": 1
          },
          {
            "id": "Z1",
            "version": 1
          }
        ],
        "title": "Relation enters a zone: F1, Z1",
        "description": "",
        "status": "open",
        "severity": "medium",
        "tags": []
      },
      "closed": false,
      "state": "current",
      "dataHash": "1a2b3c4d5e6f7",
      "sourceHash": "7f6e5d4c3b2a1",
      "registerVersion": "4f0c1d2e3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d"
    }
  ]
}
```
**403: The deployment refuses the read: the `domain` filter names a domain it refuses, or it refuses every domain present.**

```json
{
  "error": "Reading records is not permitted."
}
```
**500: A register could not be read.**

```json
{
  "error": "a message describing what went wrong"
}
```

## `PATCH /folders/{folder}`

Renames a folder of the workspace, keeping its parent. Every model below it keeps its id, so nothing that names a model's id is affected. It refuses, and renames nothing, for the workspace root, `findings` and anything under it, a folder inside a model's directory, and a folder that holds a Metron package.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `folder` | path | yes | string | The folder, relative to the workspace root, with `/` separators. Each segment is percent-encoded and the separators stay, so `sites/north` is `/folders/sites/north`. The workspace root cannot be addressed. |

### Responses

**200: The folder is renamed. `models` lists the models below it with their new locators.**

```json
{
  "ok": true,
  "folder": "sites/north-2026",
  "models": [
    {
      "kind": "tetra",
      "id": "plant-a",
      "label": "Plant A",
      "version": 1,
      "locator": "sites/north-2026/plant-a/plant-a.tetra.yaml"
    }
  ]
}
```
**400: The name is not valid, or the folder is the workspace root or is not a valid folder.**

```json
{
  "error": "name must be one folder name of 1 to 200 characters, with no slash and not starting with a dot"
}
```
**403: The caller may not write.**

```json
{
  "error": "the editor role is required to write"
}
```
**404: No folder has this path.**

```json
{
  "error": "no folder sites/north"
}
```
**409: The folder cannot be renamed. `reason` is `reserved` for `findings` and anything under it. It is `inside-model` with the `paths` of the model whose directory holds the folder. It is `holds-packages` with the `paths` of the Metron packages below it. It is `exists` when the destination is taken.**

```json
{
  "error": "sites/north is inside the directory of the model sites/north/plant-a.tetra.yaml",
  "reason": "inside-model",
  "paths": [
    "sites/north/plant-a.tetra.yaml"
  ]
}
```

## `DELETE /folders/{folder}`

Removes a folder of the workspace. It refuses, and removes nothing, while the folder holds any file, which includes every model, finding register and package file, so a caller removes or moves those first. A tree of empty folders is removed whole. It also refuses the workspace root, `findings` and anything under it, and a folder inside a model's directory.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `folder` | path | yes | string | The folder, relative to the workspace root, with `/` separators. Each segment is percent-encoded and the separators stay, so `sites/north` is `/folders/sites/north`. The workspace root cannot be addressed. |

### Responses

**200: The folder is removed.**

```json
{
  "ok": true
}
```
**400: The folder is the workspace root or is not a valid folder.**

```json
{
  "error": "the workspace root cannot be renamed, moved or deleted"
}
```
**403: The caller may not write.**

```json
{
  "error": "the editor role is required to write"
}
```
**404: No folder has this path.**

```json
{
  "error": "no folder sites/north"
}
```
**409: The folder cannot be removed. `reason` is `not-empty` with the `paths` of the files it holds (up to 50), or `reserved`. It is `inside-model` with the `paths` of the model whose directory holds the folder.**

```json
{
  "error": "sites/old still holds files: sites/old/readme.txt",
  "reason": "not-empty",
  "paths": [
    "sites/old/readme.txt"
  ]
}
```

## `POST /folders/{folder}/move`

Re-files a folder of the workspace under another folder, keeping its name. Every model below it keeps its id. It refuses, and moves nothing, for the workspace root, `findings` and anything under it, a folder inside a model's directory, a folder that holds a Metron package, a destination under `findings` or inside a model's directory, and a folder holding a model that reads a file outside the folder.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `folder` | path | yes | string | The folder, relative to the workspace root, with `/` separators. Each segment is percent-encoded and the separators stay, so `sites/north` is `/folders/sites/north`. The workspace root cannot be addressed. |

### Responses

**200: The folder is moved. `models` lists the models below it with their new locators.**

```json
{
  "ok": true,
  "folder": "archive/2026/north",
  "models": [
    {
      "kind": "tetra",
      "id": "plant-a",
      "label": "Plant A",
      "version": 1,
      "locator": "archive/2026/north/plant-a/plant-a.tetra.yaml"
    }
  ]
}
```
**400: The parent is not valid, or the folder is the workspace root or is not a valid folder.**

```json
{
  "error": "a folder cannot move into itself"
}
```
**403: The caller may not write.**

```json
{
  "error": "the editor role is required to write"
}
```
**404: No folder has this path.**

```json
{
  "error": "no folder sites/north"
}
```
**409: The folder cannot be moved. `reason` is `reserved` for `findings` and anything under it. It is `inside-model` with the `paths` of the model whose directory holds the folder. It is `holds-packages` with the `paths` of the Metron packages below it. It is `exists` when the destination is taken. It is `external-files` with the `paths` of the files that a model reads outside the folder.**

```json
{
  "error": "sites/north holds models that read files outside it: shared/zones.yaml",
  "reason": "external-files",
  "paths": [
    "shared/zones.yaml"
  ]
}
```
