## `GET /models`

Lists the models the server serves through the model routes: tetra models by default, or the package kinds a Metron server is configured for. Use `/discovery/models` for every kind. A Metron package id can appear more than once, once per version; address one with the `version` parameter on the model routes.

### Responses

**200 — The served models in the workspace.**

```json
[
  {
    "kind": "tetra",
    "id": "plant-a",
    "locator": "example/tetra.yaml"
  }
]
```

## `GET /models/{id}`

Returns every file of the model and the content-hash version of that read. An at parameter other than the current version answers 410.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `at` | query | no | string | A version to read. Only the current version is supported. |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match, such as a tetra model. |

### Responses

**200 — The model files and their version.**

```json
{
  "version": "9f2c...",
  "files": {
    "blockdiagram.tetra.yaml": "blockdiagram:\n  meta:\n    id: bd-plant-a\n"
  }
}
```
**400 — `version` is not a positive integer.**

```json
{
  "error": "version must be a positive integer: abc"
}
```
**404 — No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409 — The id names more than one model. The body lists them as candidates; repeat the request with `version`.**

```json
{
  "error": "model id pkg-a names 2 models; address one with ?version=<n>",
  "reason": "ambiguous",
  "candidates": [
    {
      "kind": "metron-rules",
      "id": "pkg-a",
      "version": 1
    },
    {
      "kind": "metron-rules",
      "id": "pkg-a",
      "version": 2
    }
  ]
}
```
**410 — at names a version other than the current one.**

```json
{
  "error": "historical model versions are not supported"
}
```

## `GET /models/{id}/versions`

This route returns exactly one entry, for the current filesystem state, with an empty author. A non-numeric limit behaves as limit=0.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `limit` | query | no | integer |  |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match, such as a tetra model. |

### Responses

**200 — The one current version.**

```json
[
  {
    "id": "9f2c...",
    "message": "Current filesystem state",
    "author": "",
    "at": "2026-09-27T00:00:00.000Z"
  }
]
```
**400 — `version` is not a positive integer.**

```json
{
  "error": "version must be a positive integer: abc"
}
```
**404 — No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409 — The id names more than one model. The body lists them as candidates; repeat the request with `version`.**

```json
{
  "error": "model id pkg-a names 2 models; address one with ?version=<n>",
  "reason": "ambiguous",
  "candidates": [
    {
      "kind": "metron-rules",
      "id": "pkg-a",
      "version": 1
    },
    {
      "kind": "metron-rules",
      "id": "pkg-a",
      "version": 2
    }
  ]
}
```

## `POST /models/{id}/commits`

Applies changes only when baseVersion matches the current version, so a stale write answers 409 with the current version and the paths it touched, instead of applying a lost update.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match, such as a tetra model. |

### Responses

**200 — The commit applied.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400 — `version` is not a positive integer, or a change path is outside the model directory.**

```json
{
  "error": "version must be a positive integer: abc"
}
```
**403 — The write could not be applied, or the caller is authenticated as a reader. A refused reader gets `{ "error": "the editor role is required to write" }` instead of the body below, and only when the server requires authentication.**

```json
{
  "ok": false,
  "reason": "forbidden"
}
```
**404 — No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409 — baseVersion is not the current version. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model: the body then holds `error` and `candidates`.**

```json
{
  "ok": false,
  "reason": "conflict",
  "current": "9f2c...",
  "paths": [
    "devices.yaml"
  ]
}
```

## `POST /models/{id}/git/commit`

Opt-in versioning, separate from the plain commits route above: commits whatever is currently on disk under this model's own directory, and no other model's uncommitted edits. Requires the workspace to already be a git repository; author defaults to a fixed server identity when omitted.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match, such as a tetra model. |

### Responses

**200 — Either a new commit, or ok:false, reason:no-changes when nothing under this model's directory changed since the last commit.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400 — The workspace is not a git repository (reason: not-a-repo). A malformed `version` also answers 400 with an `error` body.**

```json
{
  "ok": false,
  "reason": "not-a-repo"
}
```
**403 — The caller is authenticated as a reader. Present only when the server requires authentication.**

```json
{
  "error": "the editor role is required to write"
}
```
**404 — No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409 — The id names more than one model. The body lists them as candidates; repeat the request with `version`.**

```json
{
  "error": "model id pkg-a names 2 models; address one with ?version=<n>",
  "reason": "ambiguous",
  "candidates": [
    {
      "kind": "metron-rules",
      "id": "pkg-a",
      "version": 1
    },
    {
      "kind": "metron-rules",
      "id": "pkg-a",
      "version": 2
    }
  ]
}
```

## `GET /models/{id}/git/history`

Newest first. Empty when the model has never been committed through the git/commit route above.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `limit` | query | no | integer |  |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match, such as a tetra model. |

### Responses

**200 — This model's commit history.**

```json
[
  {
    "id": "a81eb2c1...",
    "message": "Rename device 1",
    "author": "Sightline Storage Server <storage-server@localhost>",
    "at": "2026-09-27T00:00:00+00:00"
  }
]
```
**400 — `version` is not a positive integer.**

```json
{
  "error": "version must be a positive integer: abc"
}
```
**404 — No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409 — The id names more than one model. The body lists them as candidates; repeat the request with `version`.**

```json
{
  "error": "model id pkg-a names 2 models; address one with ?version=<n>",
  "reason": "ambiguous",
  "candidates": [
    {
      "kind": "metron-rules",
      "id": "pkg-a",
      "version": 1
    },
    {
      "kind": "metron-rules",
      "id": "pkg-a",
      "version": 2
    }
  ]
}
```

## `GET /models/{id}/git/at/{version}`

version is a commit sha from the history route above. Answers 404 when it does not resolve to a real commit for this model.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `version` | path | yes | string | A commit sha from the history route. Not the package version. |
| `version` | query | no | integer | The package version (`meta.version`) of the model to address, as with the other model routes. Not the commit `version` in the path. |

### Responses

**200 — The model's files at that revision.**

```json
{
  "version": "a81eb2c1...",
  "files": {
    "devices.yaml": "devices:\n  - id: dev-1\n"
  }
}
```
**400 — `version` is not a positive integer.**

```json
{
  "error": "version must be a positive integer: abc"
}
```
**404 — No model has this id, or version does not resolve to a real commit for it. It also answers 404 when `version` matches no served model with this id.**

```json
{
  "error": "version not found: not-a-real-sha"
}
```
**409 — The id names more than one model. The body lists them as candidates; repeat the request with `version`.**

```json
{
  "error": "model id pkg-a names 2 models; address one with ?version=<n>",
  "reason": "ambiguous",
  "candidates": [
    {
      "kind": "metron-rules",
      "id": "pkg-a",
      "version": 1
    },
    {
      "kind": "metron-rules",
      "id": "pkg-a",
      "version": 2
    }
  ]
}
```

## `POST /models/{id}/git/pull`

A merge conflict always ends with the merge aborted, so the working tree is clean after any outcome other than ok:true. Resolving a conflict is not supported.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match, such as a tetra model. |

### Responses

**200 — The merge completed (a plain git merge fast-forwards on its own when there is no local divergence).**

```json
{
  "ok": true
}
```
**400 — The workspace is not a git repository, or has no configured remote. A malformed `version` also answers 400 with an `error` body.**

```json
{
  "ok": false,
  "reason": "no-remote"
}
```
**403 — The caller is authenticated as a reader. Present only when the server requires authentication.**

```json
{
  "error": "the editor role is required to write"
}
```
**404 — No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409 — A real merge conflict. The merge was aborted; the working tree is unchanged. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model: the body then holds `error` and `candidates`.**

```json
{
  "ok": false,
  "reason": "conflict",
  "detail": "devices.yaml"
}
```

## `POST /models/{id}/git/push`

A non-fast-forward rejection (the remote has moved on) answers 409 with reason:conflict; pull first.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match, such as a tetra model. |

### Responses

**200 — The push succeeded.**

```json
{
  "ok": true
}
```
**400 — The workspace is not a git repository, or has no configured remote. A malformed `version` also answers 400 with an `error` body.**

```json
{
  "ok": false,
  "reason": "no-remote"
}
```
**403 — The caller is authenticated as a reader. Present only when the server requires authentication.**

```json
{
  "error": "the editor role is required to write"
}
```
**404 — No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409 — The remote has diverged (non-fast-forward). Pull first. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model: the body then holds `error` and `candidates`.**

```json
{
  "ok": false,
  "reason": "conflict",
  "detail": "remote has diverged; pull first"
}
```

## `GET /models/{id}/data-slices`

Reads the `dataSlices` list from the model manifest. An app serves only the collections it supports. Items are app-typed objects with a string `id`. The route answers 404 for a collection that the server's app does not serve.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match. |

### Responses

**200 — The current list and the model's content-hash version.**

```json
{
  "version": "9f2c...",
  "items": [
    {
      "id": "scada",
      "name": "SCADA",
      "filter": "item.tags |> hasAny(\"scada\")"
    }
  ]
}
```
**400 — `version` is not a positive integer.**

```json
{
  "error": "version must be a positive integer: abc"
}
```
**404 — No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409 — `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

```json
{
  "ok": false,
  "reason": "conflict",
  "current": "a81e..."
}
```

## `PUT /models/{id}/data-slices/{itemId}`

Replaces the item with this `id`, or adds it. Sorted by `id` in the manifest. The `item.id` must equal `itemId`. The server validates the whole manifest before it writes; a failure answers 400 and the file is unchanged.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `itemId` | path | yes | string | The `id` of the item. |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match. |

### Responses

**200 — The change applied. `version` is the new content-hash version, usable as `baseVersion` here and on the commits route.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400 — `item.id` is missing or differs from `itemId`, or the manifest fails validation.**

```json
{
  "error": "item.id must match the itemId in the path"
}
```
**404 — No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409 — `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

```json
{
  "ok": false,
  "reason": "conflict",
  "current": "a81e..."
}
```

## `DELETE /models/{id}/data-slices/{itemId}`

Removes the item with this `id`. An id that is not in the list changes nothing and still answers 200.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `itemId` | path | yes | string | The `id` of the item. |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match. |
| `baseVersion` | query | no | string | The version the client last read. A different current version answers 409 and writes nothing. Omit it to write over the current state. |

### Responses

**200 — The change applied. `version` is the new content-hash version, usable as `baseVersion` here and on the commits route.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400 — The manifest fails validation after the removal.**

```json
{
  "error": "Invalid package manifest after template write"
}
```
**404 — No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409 — `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

```json
{
  "ok": false,
  "reason": "conflict",
  "current": "a81e..."
}
```

## `GET /models/{id}/query-templates`

Reads the `queryTemplates` list from the model manifest. An app serves only the collections it supports. Items are app-typed objects with a string `id`. The route answers 404 for a collection that the server's app does not serve.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match. |

### Responses

**200 — The current list and the model's content-hash version.**

```json
{
  "version": "9f2c...",
  "items": [
    {
      "id": "tpl-1",
      "name": "Template",
      "program": [
        "model |> add()"
      ]
    }
  ]
}
```
**400 — `version` is not a positive integer.**

```json
{
  "error": "version must be a positive integer: abc"
}
```
**404 — No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409 — `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

```json
{
  "ok": false,
  "reason": "conflict",
  "current": "a81e..."
}
```

## `PUT /models/{id}/query-templates`

Writes the list in the order sent, so a client can reorder it. The server validates the whole manifest first; a manifest that fails validation answers 400 and the file is unchanged.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match. |

### Responses

**200 — The change applied. `version` is the new content-hash version, usable as `baseVersion` here and on the commits route.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400 — `items` is not an array, or the manifest fails validation.**

```json
{
  "error": "items must be an array"
}
```
**404 — No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409 — `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

```json
{
  "ok": false,
  "reason": "conflict",
  "current": "a81e..."
}
```

## `PUT /models/{id}/query-templates/{itemId}`

Replaces the item with this `id`, or adds it. Kept in the order written. The `item.id` must equal `itemId`. The server validates the whole manifest before it writes; a failure answers 400 and the file is unchanged.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `itemId` | path | yes | string | The `id` of the item. |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match. |

### Responses

**200 — The change applied. `version` is the new content-hash version, usable as `baseVersion` here and on the commits route.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400 — `item.id` is missing or differs from `itemId`, or the manifest fails validation.**

```json
{
  "error": "item.id must match the itemId in the path"
}
```
**404 — No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409 — `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

```json
{
  "ok": false,
  "reason": "conflict",
  "current": "a81e..."
}
```

## `DELETE /models/{id}/query-templates/{itemId}`

Removes the item with this `id`. An id that is not in the list changes nothing and still answers 200.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `itemId` | path | yes | string | The `id` of the item. |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match. |
| `baseVersion` | query | no | string | The version the client last read. A different current version answers 409 and writes nothing. Omit it to write over the current state. |

### Responses

**200 — The change applied. `version` is the new content-hash version, usable as `baseVersion` here and on the commits route.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400 — The manifest fails validation after the removal.**

```json
{
  "error": "Invalid package manifest after template write"
}
```
**404 — No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409 — `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

```json
{
  "ok": false,
  "reason": "conflict",
  "current": "a81e..."
}
```

## `GET /models/{id}/aggregations`

Reads the `aggregations` list from the model manifest. An app serves only the collections it supports. Items are app-typed objects with a string `id`. The route answers 404 for a collection that the server's app does not serve.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match. |

### Responses

**200 — The current list and the model's content-hash version.**

```json
{
  "version": "9f2c...",
  "items": [
    {
      "id": "agg-1",
      "name": "First",
      "config": {}
    }
  ]
}
```
**400 — `version` is not a positive integer.**

```json
{
  "error": "version must be a positive integer: abc"
}
```
**404 — No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409 — `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

```json
{
  "ok": false,
  "reason": "conflict",
  "current": "a81e..."
}
```

## `PUT /models/{id}/aggregations/{itemId}`

Replaces the item with this `id`, or adds it. Kept in the order written. The `item.id` must equal `itemId`. The server validates the whole manifest before it writes; a failure answers 400 and the file is unchanged.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `itemId` | path | yes | string | The `id` of the item. |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match. |

### Responses

**200 — The change applied. `version` is the new content-hash version, usable as `baseVersion` here and on the commits route.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400 — `item.id` is missing or differs from `itemId`, or the manifest fails validation.**

```json
{
  "error": "item.id must match the itemId in the path"
}
```
**404 — No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409 — `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

```json
{
  "ok": false,
  "reason": "conflict",
  "current": "a81e..."
}
```

## `DELETE /models/{id}/aggregations/{itemId}`

Removes the item with this `id`. An id that is not in the list changes nothing and still answers 200.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `itemId` | path | yes | string | The `id` of the item. |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match. |
| `baseVersion` | query | no | string | The version the client last read. A different current version answers 409 and writes nothing. Omit it to write over the current state. |

### Responses

**200 — The change applied. `version` is the new content-hash version, usable as `baseVersion` here and on the commits route.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400 — The manifest fails validation after the removal.**

```json
{
  "error": "Invalid package manifest after template write"
}
```
**404 — No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409 — `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

```json
{
  "ok": false,
  "reason": "conflict",
  "current": "a81e..."
}
```

## `POST /models/{id}/template-files`

Writes an empty template file at `relPath`, and creates any missing folders. It does not list the file in the manifest; call `template-files/add` for that. An existing file answers 409 with `reason` set to `exists` and is not changed.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match. |

### Responses

**200 — The change applied. `version` is the new content-hash version, usable as `baseVersion` here and on the commits route.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400 — `relPath` is missing, is not a string, does not end in `.yaml` or `.yml`, or points outside the model directory, or the manifest fails validation.**

```json
{
  "error": "relPath must end in .yaml or .yml: notes.txt"
}
```
**404 — No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409 — `baseVersion` is not the current version, with `current` in the body so the client can rebase. Also answers 409 with `reason` set to `exists` when a file is already at `relPath`, and with `reason` set to `ambiguous` when the id names more than one model.**

```json
{
  "ok": false,
  "reason": "conflict",
  "current": "a81e..."
}
```

## `POST /models/{id}/template-files/add`

Appends `relPath` to the manifest's `templates` list. A path already in the list changes nothing and still answers 200. The server validates the whole manifest before it writes; a failure answers 400 and the file is unchanged.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match. |

### Responses

**200 — The change applied. `version` is the new content-hash version, usable as `baseVersion` here and on the commits route.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400 — `relPath` is missing, is not a string, does not end in `.yaml` or `.yml`, or points outside the model directory, or the manifest fails validation.**

```json
{
  "error": "relPath must end in .yaml or .yml: notes.txt"
}
```
**404 — No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409 — `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

```json
{
  "ok": false,
  "reason": "conflict",
  "current": "a81e..."
}
```

## `POST /models/{id}/template-files/remove`

Removes `relPath` from the manifest's `templates` list. The file stays on disk. A path that is not in the list changes nothing and still answers 200.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match. |

### Responses

**200 — The change applied. `version` is the new content-hash version, usable as `baseVersion` here and on the commits route.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400 — `relPath` is missing, is not a string, or does not end in `.yaml` or `.yml`, or the manifest fails validation.**

```json
{
  "error": "relPath must end in .yaml or .yml: notes.txt"
}
```
**404 — No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409 — `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

```json
{
  "ok": false,
  "reason": "conflict",
  "current": "a81e..."
}
```

## `POST /models/{id}/manifest-files`

Writes an empty model file at `relPath` when none exists there, and lists it in the manifest's `files` list in the same write. A file that already exists is listed without being changed. A path already listed changes nothing.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match. |

### Responses

**200 — The change applied. `version` is the new content-hash version, usable as `baseVersion` here and on the commits route.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400 — `relPath` is missing, is not a string, does not end in `.yaml` or `.yml`, or points outside the model directory, or the manifest fails validation.**

```json
{
  "error": "relPath must end in .yaml or .yml: notes.txt"
}
```
**404 — No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409 — `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

```json
{
  "ok": false,
  "reason": "conflict",
  "current": "a81e..."
}
```

## `POST /models/{id}/manifest-files/import`

Lists an existing file in the manifest's `files` list. A path already listed changes nothing and still answers 200. A file that does not exist answers 404 with `reason` set to `file-not-found`.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match. |

### Responses

**200 — The change applied. `version` is the new content-hash version, usable as `baseVersion` here and on the commits route.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400 — `relPath` is missing, is not a string, does not end in `.yaml` or `.yml`, or points outside the model directory, or the manifest fails validation.**

```json
{
  "error": "relPath must end in .yaml or .yml: notes.txt"
}
```
**404 — No model has this id, or none has this id at the requested `version`. Also answers 404 with `reason` set to `file-not-found` when no file exists at `relPath`.**

```json
{
  "ok": false,
  "reason": "file-not-found"
}
```
**409 — `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

```json
{
  "ok": false,
  "reason": "conflict",
  "current": "a81e..."
}
```
