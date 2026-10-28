## `GET /models`

Lists the models the server serves through the model routes: tetra models by default, or the package kinds a Metron server is configured for. Use `/discovery/models` for every kind. A Metron package id can appear more than once, once per version; address one with the `version` parameter on the model routes.

### Responses

**200: The served models in the workspace.**

```json
{
  "models": [
    {
      "kind": "tetra",
      "id": "plant-a",
      "locator": "example/tetra.yaml"
    }
  ]
}
```

## `POST /models`

Creates a model or package of one kind, which the server must be configured to create, and answers the descriptor that discovery now lists for it. The model's files go directly in `location`, the folder you chose, with the manifest and every data file named for `id`, such as `<id>.tetra.yaml` and `<id>.zones.yaml`. The `id` is written to `meta.id` and `name` is the free-text label, so a long descriptive name does not lengthen the file names. An `id` is unique across every model in the workspace, whatever its kind: an `id` that another model already holds answers 409 with `reason` set to `duplicate`. The folder may hold unrelated files and models of other apps. A folder that already holds a model of the same app, or a file the new model would write, answers 409 with `reason` set to `exists`. The four Metron package kinds count as one app. Nothing is written in either case. A failed write removes the files this request wrote and leaves the folder and its other files in place. A Metron requirement package is written alone, with an empty `uses:` list and `schemeRef: default`. It opens at once on a built-in Pass and Fail scheme, and a commit that names a scheme package in `uses:` connects a shared scheme. Choose `location` from `GET /folders`, or make one with `POST /folders`. A rule package takes a `domain`, the app that runs its rules, which defaults to `metron`.

### Responses

**201: The model was created.**

```json
{
  "ok": true,
  "descriptor": {
    "kind": "tetra",
    "id": "plant-a",
    "label": "Plant A",
    "version": 1,
    "locator": "sites/plant-a.tetra.yaml"
  }
}
```
**400: The body is not JSON. `kind` is missing or not creatable on this server. `id` is missing, is not a string, or does not match the pattern. `name` is missing or blank. `location` is not a string or leaves the workspace. `domain` is not a string without whitespace or `/`.**

```json
{
  "error": "Use lowercase letters, digits and single hyphens, up to 64 characters."
}
```
**403: The caller is authenticated as a reader. This is the answer only when the server requires authentication.**

```json
{
  "error": "the editor role is required to write"
}
```
**409: Nothing was written. `reason` is `duplicate` when another model in the workspace already has this `id`, and the body repeats it as `id`. `reason` is `exists` when the folder already holds a model of the same app or a file this model would write.**

```json
{
  "ok": false,
  "reason": "duplicate",
  "id": "plant-a"
}
```
**500: A write failed. The files this request wrote were removed, and the folder and its other files are as they were.**

```json
{
  "error": "disk full"
}
```

## `GET /models/{id}`

Returns every file of the model and the content-hash version of that read. An at parameter other than the current version answers 410.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `at` | query | no | string | A version to read. Only the current version is supported. |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match, such as a tetra model. |

### Responses

**200: The model files and their version.**

```json
{
  "version": "9f2c...",
  "files": {
    "blockdiagram.tetra.yaml": "blockdiagram:\n  meta:\n    id: bd-plant-a\n"
  },
  "entry": "blockdiagram.tetra.yaml",
  "dir": "plant-a"
}
```
**400: `version` is not a positive integer.**

```json
{
  "error": "version must be a positive integer: abc"
}
```
**404: No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409: The id names more than one model. The body lists them as candidates; repeat the request with `version`.**

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
**410: at names a version other than the current one.**

```json
{
  "error": "historical model versions are not supported"
}
```

## `PATCH /models/{id}`

Changes the name a Tetra or Lamina model is listed under. Only that name changes: the id, the directory and every other field and comment of the manifest stay as they are, so nothing that names the model's id is affected. Other model kinds answer 400 with `reason: "unsupported-kind"`.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a model whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a Tetra or Lamina model. |

### Responses

**200: The model is renamed. The descriptor is the listing's entry for it.**

```json
{
  "ok": true,
  "descriptor": {
    "kind": "tetra",
    "id": "plant-a",
    "label": "Plant A (north site)",
    "version": 1,
    "locator": "sites/plant-a/plant-a.tetra.yaml"
  }
}
```
**400: The model is not a Tetra or Lamina model, or the request is not valid. The `reason` is `unsupported-kind` for a model of another kind.**

```json
{
  "error": "a metron-package model cannot be renamed, moved or deleted here"
}
```
**403: The caller may not write.**

```json
{
  "error": "the editor role is required to write"
}
```
**404: No model has this id.**

```json
{
  "error": "model not found: plant-a"
}
```

## `DELETE /models/{id}`

Removes a Tetra or Lamina model's directory and everything in it. It refuses, and removes nothing, while another file still names the model's id: a findings register, a bound Metron assessment, or a Lamina manifest that scopes to it. It also refuses when the directory is the workspace root or holds other models. A caller removes or re-points the referrers first. Other model kinds answer 400 with `reason: "unsupported-kind"`.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a model whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a Tetra or Lamina model. |

### Responses

**200: The model's directory is removed.**

```json
{
  "ok": true
}
```
**400: The model is not a Tetra or Lamina model, or the request is not valid. The `reason` is `unsupported-kind` for a model of another kind.**

```json
{
  "error": "a metron-package model cannot be renamed, moved or deleted here"
}
```
**403: The caller may not write.**

```json
{
  "error": "the editor role is required to write"
}
```
**404: No model has this id.**

```json
{
  "error": "model not found: plant-a"
}
```
**409: The model cannot be removed. `reason` is `in-use` with the `referrers`, `shared-directory` with the `paths` of the other models, or `ambiguous` with `candidates` when the id names several models.**

```json
{
  "error": "plant-a is still referenced by 1 file",
  "reason": "in-use",
  "referrers": [
    {
      "kind": "assessment",
      "id": "plant-a-assessment",
      "path": "sites/plant-a/plant-a.example-package-v1.assessment.yaml"
    }
  ]
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

**200: The one current version.**

```json
{
  "versions": [
    {
      "id": "9f2c...",
      "message": "Current filesystem state",
      "author": "",
      "at": "2026-09-27T00:00:00.000Z"
    }
  ]
}
```
**400: `version` is not a positive integer.**

```json
{
  "error": "version must be a positive integer: abc"
}
```
**404: No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409: The id names more than one model. The body lists them as candidates; repeat the request with `version`.**

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

## `GET /models/{id}/events`

Opens a server-sent event stream. The first event is `ready`, whose data is the model's current content-hash version. Each later change to the model sends a `changed` event whose data is the new version. A client that loaded an older version than `ready` names is behind. The stream stays open until the client closes it.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match, such as a tetra model. |

### Responses

**200: The event stream.**

```text
event: ready
data: 9f2c...

event: changed
data: 41ab...


```
**400: `version` is not a positive integer.**

```json
{
  "error": "version must be a positive integer: abc"
}
```
**404: No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409: The id names more than one model. The body lists them as candidates; repeat the request with `version`.**

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

Applies changes only when `baseVersion` matches the current version. A stale write answers 409 with the current version and the paths it touched, so no update is lost. The commit applies every change or none. A failed write restores every file. Two server processes that write one directory are not coordinated.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match, such as a tetra model. |

### Responses

**200: The commit applied.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400: `version` is not a positive integer, a change path is outside the model directory, or the commit would leave a Tetra model the loader refuses (the answer carries the loader message and nothing is written).**

```json
{
  "error": "version must be a positive integer: abc"
}
```
**403: The write could not be applied, or the caller is authenticated as a reader. A refused reader gets `{ "error": "the editor role is required to write" }` instead of the body below, and only when the server requires authentication.**

```json
{
  "ok": false,
  "reason": "forbidden"
}
```
**404: No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409: baseVersion is not the current version. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model: the body then holds `error` and `candidates`.**

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

## `POST /models/{id}/outputs`

Writes one file into the model's `outputs/` folder and creates the folder when it is missing. The host that holds the bytes of a PNG, a draw.io file or a generated report sends them here. The file is not part of the model, so its version does not change and no `baseVersion` applies. An existing file with the same name is replaced.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match. |

### Responses

**200: The file is written. `path` is relative to the model directory.**

```json
{
  "ok": true,
  "path": "outputs/plant-a.png"
}
```
**400: The body is not JSON. `fileName` is missing or not a bare supported file name. `data` is not base64, or the request would write outside the model directory.**

```json
{
  "error": "fileName must be a bare file name: ../x.png"
}
```
**403: The caller may not write.**

```json
{
  "error": "the editor role is required to write"
}
```
**404: No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```

## `POST /models/{id}/git/commit`

Opt-in versioning, separate from the plain commits route above: commits whatever is currently on disk under this model's own directory, and no other model's uncommitted edits. Requires the workspace to already be a git repository; author defaults to a fixed server identity when omitted.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a Metron package whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a model with one match, such as a tetra model. |

### Responses

**200: Either a new commit, or ok:false, reason:no-changes when nothing under this model's directory changed since the last commit.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400: The workspace is not a git repository (reason: not-a-repo). A malformed `version` also answers 400 with an `error` body.**

```json
{
  "ok": false,
  "reason": "not-a-repo"
}
```
**403: The caller is authenticated as a reader. Present only when the server requires authentication.**

```json
{
  "error": "the editor role is required to write"
}
```
**404: No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409: The id names more than one model. The body lists them as candidates; repeat the request with `version`.**

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

**200: This model's commit history.**

```json
{
  "versions": [
    {
      "id": "a81eb2c1...",
      "message": "Rename device 1",
      "author": "Sightline Storage Server <storage-server@localhost>",
      "at": "2026-09-27T00:00:00+00:00"
    }
  ]
}
```
**400: `version` is not a positive integer.**

```json
{
  "error": "version must be a positive integer: abc"
}
```
**404: No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409: The id names more than one model. The body lists them as candidates; repeat the request with `version`.**

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

**200: The model's files at that revision.**

```json
{
  "version": "a81eb2c1...",
  "files": {
    "devices.yaml": "devices:\n  - id: dev-1\n"
  }
}
```
**400: `version` is not a positive integer.**

```json
{
  "error": "version must be a positive integer: abc"
}
```
**404: No model has this id, or version does not resolve to a real commit for it. It also answers 404 when `version` matches no served model with this id.**

```json
{
  "error": "version not found: not-a-real-sha"
}
```
**409: The id names more than one model. The body lists them as candidates; repeat the request with `version`.**

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

**200: The merge completed (a plain git merge fast-forwards on its own when there is no local divergence).**

```json
{
  "ok": true
}
```
**400: The workspace is not a git repository, or has no configured remote. A malformed `version` also answers 400 with an `error` body.**

```json
{
  "ok": false,
  "reason": "no-remote"
}
```
**403: The caller is authenticated as a reader. Present only when the server requires authentication.**

```json
{
  "error": "the editor role is required to write"
}
```
**404: No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409: A real merge conflict. The merge was aborted; the working tree is unchanged. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model: the body then holds `error` and `candidates`.**

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

**200: The push succeeded.**

```json
{
  "ok": true
}
```
**400: The workspace is not a git repository, or has no configured remote. A malformed `version` also answers 400 with an `error` body.**

```json
{
  "ok": false,
  "reason": "no-remote"
}
```
**403: The caller is authenticated as a reader. Present only when the server requires authentication.**

```json
{
  "error": "the editor role is required to write"
}
```
**404: No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409: The remote has diverged (non-fast-forward). Pull first. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model: the body then holds `error` and `candidates`.**

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

**200: The current list and the model's content-hash version.**

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
**400: `version` is not a positive integer.**

```json
{
  "error": "version must be a positive integer: abc"
}
```
**404: No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409: `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

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

**200: The change applied. `version` is the new content-hash version, usable as `baseVersion` here and on the commits route.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400: `item.id` is missing or differs from `itemId`, or the manifest fails validation.**

```json
{
  "error": "item.id must match the itemId in the path"
}
```
**404: No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409: `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

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

**200: The change applied. `version` is the new content-hash version, usable as `baseVersion` here and on the commits route.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400: The manifest fails validation after the removal.**

```json
{
  "error": "Invalid package manifest after template write"
}
```
**404: No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409: `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

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

**200: The current list and the model's content-hash version.**

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
**400: `version` is not a positive integer.**

```json
{
  "error": "version must be a positive integer: abc"
}
```
**404: No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409: `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

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

**200: The change applied. `version` is the new content-hash version, usable as `baseVersion` here and on the commits route.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400: `items` is not an array, or the manifest fails validation.**

```json
{
  "error": "items must be an array"
}
```
**404: No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409: `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

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

**200: The change applied. `version` is the new content-hash version, usable as `baseVersion` here and on the commits route.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400: `item.id` is missing or differs from `itemId`, or the manifest fails validation.**

```json
{
  "error": "item.id must match the itemId in the path"
}
```
**404: No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409: `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

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

**200: The change applied. `version` is the new content-hash version, usable as `baseVersion` here and on the commits route.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400: The manifest fails validation after the removal.**

```json
{
  "error": "Invalid package manifest after template write"
}
```
**404: No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409: `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

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

**200: The change applied. `version` is the new content-hash version, usable as `baseVersion` here and on the commits route.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400: `relPath` is missing, is not a string, or does not end in `.yaml` or `.yml`. It points outside the model directory, or the manifest fails validation.**

```json
{
  "error": "relPath must end in .yaml or .yml: notes.txt"
}
```
**404: No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409: `baseVersion` is not the current version, with `current` in the body so the client can rebase. Also answers 409 with `reason` set to `exists` when a file is already at `relPath`. It answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

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

**200: The change applied. `version` is the new content-hash version, usable as `baseVersion` here and on the commits route.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400: `relPath` is missing, is not a string, or does not end in `.yaml` or `.yml`. It points outside the model directory, or the manifest fails validation.**

```json
{
  "error": "relPath must end in .yaml or .yml: notes.txt"
}
```
**404: No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409: `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

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

**200: The change applied. `version` is the new content-hash version, usable as `baseVersion` here and on the commits route.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400: `relPath` is missing, is not a string, or does not end in `.yaml` or `.yml`, or the manifest fails validation.**

```json
{
  "error": "relPath must end in .yaml or .yml: notes.txt"
}
```
**404: No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409: `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

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

**200: The change applied. `version` is the new content-hash version, usable as `baseVersion` here and on the commits route.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400: `relPath` is missing, is not a string, or does not end in `.yaml` or `.yml`. It points outside the model directory, or the manifest fails validation.**

```json
{
  "error": "relPath must end in .yaml or .yml: notes.txt"
}
```
**404: No model has this id, or none has this id at the requested `version`.**

```json
{
  "error": "model not found: plant-a"
}
```
**409: `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

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

**200: The change applied. `version` is the new content-hash version, usable as `baseVersion` here and on the commits route.**

```json
{
  "ok": true,
  "version": "a81e..."
}
```
**400: `relPath` is missing, is not a string, or does not end in `.yaml` or `.yml`. It points outside the model directory, or the manifest fails validation.**

```json
{
  "error": "relPath must end in .yaml or .yml: notes.txt"
}
```
**404: No model has this id, or none has this id at the requested `version`. Also answers 404 with `reason` set to `file-not-found` when no file exists at `relPath`.**

```json
{
  "ok": false,
  "reason": "file-not-found"
}
```
**409: `baseVersion` is not the current version. The body holds `current`, so the client can rebase. Also answers 409 with `reason` set to `ambiguous` when the id names more than one model.**

```json
{
  "ok": false,
  "reason": "conflict",
  "current": "a81e..."
}
```

## `GET /models/{modelId}/findings/{findingId}`

Answers one record with its derived state and the version of its register. The record id must belong to the model in the path. The grant the read needs is the one of the record's domain, so a licence that holds only the Tetra grant reads `tetra` records and is refused `metron` ones.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The model id (`blockdiagram.meta.id`). |
| `findingId` | path | yes | string | The record id, percent-encoded as one path segment. |

### Responses

**200: The record.**

```json
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
```
**403: The deployment refuses the read.**

```json
{
  "error": "Reading records is not permitted."
}
```
**404: No record has that id in that model.**

```json
{
  "error": "record not found: example-rules/cross-zone-relation:example-grid/relationId=F9,zoneId=Z9"
}
```

## `PATCH /models/{modelId}/findings/{findingId}`

Edits a record's `title`, `state` and person-layer values, and, on a record of a manual rule, its `data`. Each field touched carries a base in `base`, the hash of the value the caller read. So two people editing different fields never conflict. A run that rewrote `data` does not conflict with a person editing `person`. A field whose current value is not the one read answers 409 `conflict` naming the `fields`. A `person` field set to `null` is cleared, which returns the layer below. A change appends one `changed` entry to the journal that carries the values the record now shows. The fields `status`, `severity`, `description`, `tags`, `links` and `baseVersion` are not accepted. Each answers 400 and names the field that replaces it.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The model id (`blockdiagram.meta.id`). |
| `findingId` | path | yes | string | The record id, percent-encoded as one path segment. |

### Responses

**200: The record after the edit.**

```json
{
  "ok": true,
  "version": "5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f",
  "record": {
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
      "status": "reviewed",
      "severity": "medium",
      "tags": []
    },
    "closed": false,
    "state": "current",
    "dataHash": "1a2b3c4d5e6f7",
    "sourceHash": "7f6e5d4c3b2a1",
    "registerVersion": "4f0c1d2e3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d"
  }
}
```
**400: The request is refused: the body is invalid, or `data` or `links` was sent for a record of a generator rule. Nothing was written.**

```json
{
  "ok": false,
  "reason": "invalid",
  "error": "Only a manual record has a data and links a person can edit."
}
```
**403: The deployment refuses the write, or the register could not be written. Nothing was written.**

```json
{
  "ok": false,
  "reason": "forbidden",
  "error": "Writing records is not permitted."
}
```
**404: No record has that id in that model.**

```json
{
  "ok": false,
  "reason": "not-found",
  "error": "record not found: example-rules/cross-zone-relation:example-grid/relationId=F9,zoneId=Z9"
}
```
**409: A field changed since the caller read it. Nothing was written.**

```json
{
  "ok": false,
  "reason": "conflict",
  "error": "A field changed since it was read.",
  "fields": [
    "person.severity"
  ]
}
```

## `GET /models/{modelId}/findings/{findingId}/journal`

Answers the journal entries of one record, oldest first. Each entry carries the full `data` and `links` it set, so a reader never replays differences. The grant the read needs is the one of the record's domain, so a licence that holds only the Tetra grant reads `tetra` records and is refused `metron` ones.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The model id (`blockdiagram.meta.id`). |
| `findingId` | path | yes | string | The record id, percent-encoded as one path segment. |

### Responses

**200: The journal entries of the record.**

```json
{
  "entries": [
    {
      "record": "example-rules/cross-zone-relation:example-grid/relationId=F1,zoneId=Z1",
      "at": "2026-10-10T01:00:00Z",
      "by": "user-1",
      "run": "run-2026-10-10T01:00:00Z",
      "rules": "example-rules@1",
      "kind": "created",
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
      ]
    }
  ]
}
```
**403: The deployment refuses the read.**

```json
{
  "error": "Reading records is not permitted."
}
```
**404: No record has that id in that model.**

```json
{
  "error": "record not found: example-rules/cross-zone-relation:example-grid/relationId=F9,zoneId=Z9"
}
```

## `GET /models/{modelId}/findings/{findingId}/usages`

Lists the causes that link the record through a `record` link on their vector, across every Lamina model in the workspace. Nothing is stored on the record itself. An unlinked record answers an empty list. The grant the read needs is the one of the record's domain, so a licence that holds only the Tetra grant reads `tetra` records and is refused `metron` ones.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The model id (`blockdiagram.meta.id`). |
| `findingId` | path | yes | string | The record id, percent-encoded as one path segment. |

### Responses

**200: The linking causes.**

```json
{
  "usages": [
    {
      "modelId": "lam-1",
      "causeId": "C001"
    }
  ]
}
```
**403: The deployment refuses the read.**

```json
{
  "error": "Reading records is not permitted."
}
```
**404: No record has that id in that model.**

```json
{
  "error": "record not found: example-rules/cross-zone-relation:example-grid/relationId=F9,zoneId=Z9"
}
```

## `GET /models/{modelId}/findings/{findingId}/members`

Lists the members of a Metron finding. A member is an assessment record named by a manual `record` link whose `instance` is the assessment id. Each answer carries the drift of its link, derived now from the assessment register and never stored: `current`, `data-changed` when the assessed outcome moved, `source-changed` when the entity or requirement moved, `both-changed`, `new` for a member that was never accepted, `closed`, `missing` when the assessment or its record is absent, or `unreadable` when its register cannot be read. A record that has no member links answers an empty list. The grant the read needs is the one of the record's domain.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The model id (`blockdiagram.meta.id`). |
| `findingId` | path | yes | string | The record id, percent-encoded as one path segment. |

### Responses

**200: The members and their drift.**

```json
{
  "members": [
    {
      "assessmentId": "example-assessment",
      "recordId": "example-requirements/R-1:example-grid/entityId=Z9",
      "kind": "R-1",
      "entityId": "Z9",
      "drift": "data-changed",
      "packageId": "example-requirements",
      "requirementVersion": 2,
      "entityVersion": 3
    }
  ]
}
```
**403: The deployment refuses the read.**

```json
{
  "error": "Reading records is not permitted."
}
```
**404: No record has that id in that model.**

```json
{
  "error": "record not found: example-rules/cross-zone-relation:example-grid/relationId=F9,zoneId=Z9"
}
```

## `POST /models/{modelId}/findings`

Creates a record under a manual rule of a rule package and appends a `created` entry to the journal. The rule must be manual; a rule with a program creates its records by running. `description`, `severity` and `tags` are stored in the `person` layer. A record with the same key already in the register answers 409 with `reason: "duplicate"` and the existing `id`.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The model id (`blockdiagram.meta.id`). |

### Responses

**201: The record was created.**

```json
{
  "ok": true,
  "version": "5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f",
  "record": {
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
}
```
**400: The request is refused: the body is invalid, or the package or rule is unknown, or the rule is not manual. Nothing was written.**

```json
{
  "ok": false,
  "reason": "invalid",
  "error": "Rule cross-zone-relation is not a manual rule, so a person cannot create its records."
}
```
**403: The deployment refuses the write, or the register could not be written. Nothing was written.**

```json
{
  "ok": false,
  "reason": "forbidden",
  "error": "Writing records is not permitted."
}
```
**404: No Tetra model has that id.**

```json
{
  "ok": false,
  "reason": "not-found",
  "error": "model not found: nobody"
}
```
**409: The register changed since the caller read it (`conflict`), or a record with that key exists (`duplicate`). Nothing was written.**

```json
{
  "ok": false,
  "reason": "duplicate",
  "error": "A record with that key already exists.",
  "id": "example-rules/cross-zone-relation:example-grid/relationId=F1,zoneId=Z1"
}
```

## `GET /models/{modelId}/rule-packages`

Lists the rule packages of one domain for the model, each with its rules grouped by the fragment file that holds them, and a `baseVersion` that a rule write carries back. For the `tetra` domain these are the packages the model names in its manifest `uses:`; for `metron` they are the packages named by the requirement packages bound to the model. A `uses:` entry that matches no package, or more than one, appears in `diagnostics` and contributes no package. For the `tetra` domain the answer also carries the manifest's `uses` entries as written, including one that resolves to nothing, and a `usesVersion` that a write of the list carries back.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The model id (`blockdiagram.meta.id`). |
| `domain` | query | no | string | The rule domain. `tetra` (the default) lists the packages the Tetra manifest names in `uses:`; `metron` lists the rule packages that the requirement packages bound to the model through its assessments name in `uses:`. |

### Responses

**200: The packages and the diagnostics.**

```json
{
  "packages": [
    {
      "id": "example-rules",
      "name": "Example rules",
      "version": 1,
      "baseVersion": "3f2a9c0d",
      "files": [
        {
          "path": "rules/example.rules.yaml",
          "rules": [
            {
              "ruleId": "example-rule",
              "title": "Example rule",
              "mode": "generator",
              "input": "model",
              "hide": false,
              "program": [
                "model |> where(\"true\")"
              ]
            }
          ]
        }
      ]
    }
  ],
  "diagnostics": []
}
```
**403: The read was refused.**
**404: No such model.**

## `PUT /models/{modelId}/rule-packages/{packageId}/rules/{ruleId}`

Replaces one rule in the fragment file that holds it. Only that rule's text changes. `baseVersion` is the value the package read answered; a package that changed on disk since answers 409 with the current version and nothing is written. The body's `rule.ruleId` must equal the path's `ruleId`: a rule's id is fixed once created. A rule whose `dependsOn` closes a cycle, directly or through another rule of the model, is refused with 400 and nothing is written. The write is recorded in the audit log.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The model id (`blockdiagram.meta.id`). |
| `packageId` | path | yes | string | The rule package id. |
| `ruleId` | path | yes | string | The rule id. |
| `domain` | query | no | string | The domain whose packages the rule is looked up in. Absent means `tetra`. |

### Responses

**200: The rule was written.**

```json
{
  "ok": true,
  "baseVersion": "9c41d7e2"
}
```
**400: The body is not `{ rule, baseVersion }`, the rule is invalid, its id differs from the path, or its `dependsOn` closes a cycle.**
**403: The write was refused.**
**404: No such model, package or rule.**
**409: The package changed since it was read.**

## `PUT /models/{modelId}/uses`

Replaces the whole `uses:` list of the Tetra model's manifest with the list in the body. Only the `uses:` entries change: the manifest's comments and other keys keep their text. An entry that is already listed stays as it is. An entry that is new, or whose `version` changed, must resolve to exactly one installed package. An entry that would contribute no rule is refused with 400. Entries keep their order, and new entries come last. `baseVersion` is the `usesVersion` the package read answered; a manifest that changed on disk since answers 409 with the current version and nothing is written. The write is recorded in the audit log.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The model id (`blockdiagram.meta.id`). |

### Responses

**200: The list was written, or it already read as given.**

```json
{
  "ok": true,
  "uses": [
    {
      "kind": "rules",
      "id": "example-rules",
      "version": 1
    }
  ],
  "usesVersion": "9c41d7e2"
}
```
**400: The body is not `{ uses, baseVersion }`, an entry is malformed, or an id is listed twice. A new or re-pinned entry matches no installed package or more than one.**
**403: The write was refused.**
**404: No such model.**
**409: The manifest changed since it was read.**

## `POST /models/{modelId}/finding-runs`

Runs the generator rules of one domain and reconciles each package's output against that package's register. A new key creates a record, changed data or links updates it, a key that is no longer emitted closes, and a closed key that is emitted again returns. A run never deletes a record and never changes the fields a person owns. A package whose programs report an error is skipped, so a broken rule cannot close its records. A rule with `input: records` reads the records of the rules named in its `dependsOn`, and the run executes rules in dependency order, so a rollup sees the records its dependencies create or change in the same run. A rule that depends on a rule that cannot run does not run, and its package is skipped. With `dryRun: true` the route computes the same entries and writes nothing. Every real run appends to the journal only when something changed, so a second run of an unchanged model writes nothing. A `uses:` entry that matches no package, or more than one, appears in `diagnostics` with an `error` severity and no `ruleId`; it contributes no package, so no record closes. The `tetra` domain (the default) runs the rule packages the model lists in its manifest `uses:`; a model with no `uses:` runs no rule. The `metron` domain runs the rule packages that the requirement packages bound to the model name in `uses:`, over the assessed pairs and attributes of every binding on the model, and a model with no binding runs no rule. A run also re-resolves the links of manual records and appends a `changed` entry, with the data unchanged, when a version or hash moved; a link under a scope the run does not hold is left as stored and reported as a warning. A dry run whose candidate `rule` has `mode: transform` answers `transform`, the number of rows its `save(result)` took or the value it took; a transform creates no record either.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The model id (`blockdiagram.meta.id`). |

### Responses

**200: The run completed.**

```json
{
  "ok": true,
  "dryRun": true,
  "summary": {
    "created": 1,
    "changed": 0,
    "closed": 0,
    "returned": 0,
    "skippedPackages": 0
  },
  "entries": [
    {
      "record": "example-rules/cross-zone-relation:example-grid/relationId=F1,zoneId=Z1",
      "at": "2026-10-10T01:00:00Z",
      "by": "user-1",
      "run": "run-2026-10-10T01:00:00Z",
      "rules": "example-rules@1",
      "kind": "created",
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
      ]
    }
  ],
  "diagnostics": []
}
```
**400: The body is invalid, or the model could not be loaded. Nothing was written.**

```json
{
  "ok": false,
  "reason": "invalid",
  "error": "The body is {} or { \"dryRun\": boolean }."
}
```
**403: The deployment refuses the write, or the register could not be written. Nothing was written.**

```json
{
  "ok": false,
  "reason": "forbidden",
  "error": "Writing records is not permitted."
}
```
**404: No Tetra model has that id.**

```json
{
  "ok": false,
  "reason": "not-found",
  "error": "model not found: nobody"
}
```
**409: The register changed since the caller read it. Nothing was written; retry from `current`.**

```json
{
  "ok": false,
  "reason": "conflict",
  "error": "The register changed since it was read.",
  "current": "9d8c7b6a5f4e3d2c1b0a9f8e7d6c5b4a39281706f5e4d3c2b1a0f9e8d7c6b5a",
  "paths": [
    "registers/example.jsonl",
    "registers/example-journal.jsonl"
  ]
}
```

## `GET /models/{modelId}/diagnostics`

Reconciles the diagnostic generators of the rule packages the model uses, in memory, against each package's stored register, and answers the open diagnostics. A diagnostic generator is a `generator` rule with `hide: true` whose records carry a `message` and a `severity`. The route computes the set from the model as saved, so opening a model never writes. It reads the registers, writes no register or journal and records no audit event. A person's commit of the model journals the same set, so the answer after a commit equals the register. Each row names the record (`recordId`) and the entity it is about (`entityKind`, `entityId`) and carries no file or line. Four cases appear in `problems`. A rule's program fails. A diagnostic generator depends on a rule outside the diagnostic generators. A register is damaged. A `uses:` entry resolved to no package. The route answers 200 with empty lists when the model uses no diagnostic generator.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The model id (`blockdiagram.meta.id`). |
| `domain` | query | no | string | The rule domain. `tetra` (the default) runs the packages the Tetra manifest names in `uses:`. |

### Responses

**200: The diagnostics and the problems.**

```json
{
  "diagnostics": [
    {
      "ruleId": "channel-without-path",
      "packageId": "example-quality",
      "severity": "warning",
      "message": "Channel C-014 has no path",
      "entityId": "C-014",
      "entityKind": "channel",
      "recordId": "example-quality/channel-without-path:example-grid/entityId=C-014"
    }
  ],
  "problems": []
}
```
**400: The domain is not `tetra` or `metron`, or the model could not be loaded.**
**403: The caller may not read.**
**404: No model has this id.**

## `POST /models/{id}/move`

Re-files a Tetra or Lamina model's whole directory under another folder of the workspace, keeping its directory name. The id does not change, so nothing that names the model's id is affected. A bound assessment that sits in the directory moves with it. It refuses, and moves nothing, when the target directory exists, when the model's directory is the workspace root or holds other models, or when the model reads a file outside its own directory. Other model kinds answer 400 with `reason: "unsupported-kind"`.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string |  |
| `version` | query | no | integer | The package version (`meta.version`) to address, for a model whose id is shared by several versions. Not the content-hash version that responses carry. Omit it for a Tetra or Lamina model. |

### Responses

**200: The model is moved. The descriptor carries the new `locator`.**

```json
{
  "ok": true,
  "descriptor": {
    "kind": "tetra",
    "id": "plant-a",
    "label": "Plant A",
    "version": 1,
    "locator": "archive/2026/plant-a/plant-a.tetra.yaml"
  }
}
```
**400: The model is not a Tetra or Lamina model, or the request is not valid. The `reason` is `unsupported-kind` for a model of another kind.**

```json
{
  "error": "a metron-package model cannot be renamed, moved or deleted here"
}
```
**403: The caller may not write.**

```json
{
  "error": "the editor role is required to write"
}
```
**404: No model has this id.**

```json
{
  "error": "model not found: plant-a"
}
```
**409: The model is not moved. `reason` is `exists`, `shared-directory` or `external-files` (with the `paths` involved), or `ambiguous` with `candidates` when the id names several models.**

```json
{
  "error": "plant-a reads files outside its directory: ../shared/zones.yaml",
  "reason": "external-files",
  "paths": [
    "../shared/zones.yaml"
  ]
}
```

## `POST /models/{modelId}/findings/{findingId}/links`

Adds a link the record keeps until it is removed. The link is manual: its pin moves only on Accept. A link that names the same scope, instance and id answers 409 `duplicate`, and a `record` link that would close a cycle of links answers 400. Appends a `changed` entry to the journal.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The model id (`blockdiagram.meta.id`). |
| `findingId` | path | yes | string | The record id, percent-encoded as one path segment. |

### Responses

**201: The record after the write.**

```json
{
  "ok": true,
  "version": "5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f",
  "record": {
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
      "status": "reviewed",
      "severity": "medium",
      "tags": []
    },
    "closed": false,
    "state": "current",
    "dataHash": "1a2b3c4d5e6f7",
    "sourceHash": "7f6e5d4c3b2a1",
    "registerVersion": "4f0c1d2e3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d"
  }
}
```
**400: The request is refused: the body is invalid, or `data` or `links` was sent for a record of a generator rule. Nothing was written.**

```json
{
  "ok": false,
  "reason": "invalid",
  "error": "Only a manual record has a data and links a person can edit."
}
```
**403: The deployment refuses the write, or the register could not be written. Nothing was written.**

```json
{
  "ok": false,
  "reason": "forbidden",
  "error": "Writing records is not permitted."
}
```
**404: No record has that id in that model.**

```json
{
  "ok": false,
  "reason": "not-found",
  "error": "record not found: example-rules/cross-zone-relation:example-grid/relationId=F9,zoneId=Z9"
}
```
**409: The record already has that link.**

```json
{
  "ok": false,
  "reason": "duplicate",
  "error": "The record already has that link."
}
```

## `DELETE /models/{modelId}/findings/{findingId}/links/{linkId}`

Removes a manual link. A link a rule wrote (`auto`) answers 400, because the next run writes it again. Appends a `changed` entry to the journal.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The model id (`blockdiagram.meta.id`). |
| `findingId` | path | yes | string | The record id, percent-encoded as one path segment. |
| `linkId` | path | yes | string | The link's scope, instance and id joined by a vertical bar, percent-encoded as one path segment. |

### Responses

**200: The record after the write.**

```json
{
  "ok": true,
  "version": "5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f",
  "record": {
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
      "status": "reviewed",
      "severity": "medium",
      "tags": []
    },
    "closed": false,
    "state": "current",
    "dataHash": "1a2b3c4d5e6f7",
    "sourceHash": "7f6e5d4c3b2a1",
    "registerVersion": "4f0c1d2e3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d"
  }
}
```
**400: The request is refused: the body is invalid, or `data` or `links` was sent for a record of a generator rule. Nothing was written.**

```json
{
  "ok": false,
  "reason": "invalid",
  "error": "Only a manual record has a data and links a person can edit."
}
```
**403: The deployment refuses the write, or the register could not be written. Nothing was written.**

```json
{
  "ok": false,
  "reason": "forbidden",
  "error": "Writing records is not permitted."
}
```
**404: No record has that id in that model.**

```json
{
  "ok": false,
  "reason": "not-found",
  "error": "record not found: example-rules/cross-zone-relation:example-grid/relationId=F9,zoneId=Z9"
}
```

## `POST /models/{modelId}/findings/{findingId}/approvals`

Approves the record as it stands: copies the values its lower layers supply into `snapshot`, fills the `accepted` slot of every link from its target now, and appends an `approved` entry to the journal. `seen` carries the hashes the person saw, and a record that has changed since answers 409 `stale` with the current hashes. A link whose target is missing or cannot be read answers 400, because a pin to nothing records no state.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The model id (`blockdiagram.meta.id`). |
| `findingId` | path | yes | string | The record id, percent-encoded as one path segment. |

### Responses

**200: The record after the write.**

```json
{
  "ok": true,
  "version": "5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f",
  "record": {
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
      "status": "reviewed",
      "severity": "medium",
      "tags": []
    },
    "closed": false,
    "state": "current",
    "dataHash": "1a2b3c4d5e6f7",
    "sourceHash": "7f6e5d4c3b2a1",
    "registerVersion": "4f0c1d2e3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d"
  }
}
```
**400: The request is refused: the body is invalid, or `data` or `links` was sent for a record of a generator rule. Nothing was written.**

```json
{
  "ok": false,
  "reason": "invalid",
  "error": "Only a manual record has a data and links a person can edit."
}
```
**403: The deployment refuses the write, or the register could not be written. Nothing was written.**

```json
{
  "ok": false,
  "reason": "forbidden",
  "error": "Writing records is not permitted."
}
```
**404: No record has that id in that model.**

```json
{
  "ok": false,
  "reason": "not-found",
  "error": "record not found: example-rules/cross-zone-relation:example-grid/relationId=F9,zoneId=Z9"
}
```
**409: The record changed since the caller read it. Nothing was written.**

```json
{
  "ok": false,
  "reason": "stale",
  "error": "The record changed since it was read."
}
```

## `POST /models/{modelId}/findings/{findingId}/approvals/withdraw`

Clears the record's `snapshot`, which returns it to its live layers, and appends a `withdrawn` entry to the journal. The `accepted` pins stay. A record that is not approved answers 400.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `modelId` | path | yes | string | The model id (`blockdiagram.meta.id`). |
| `findingId` | path | yes | string | The record id, percent-encoded as one path segment. |

### Responses

**200: The record after the write.**

```json
{
  "ok": true,
  "version": "5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f",
  "record": {
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
      "status": "reviewed",
      "severity": "medium",
      "tags": []
    },
    "closed": false,
    "state": "current",
    "dataHash": "1a2b3c4d5e6f7",
    "sourceHash": "7f6e5d4c3b2a1",
    "registerVersion": "4f0c1d2e3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d"
  }
}
```
**400: The request is refused: the body is invalid, or `data` or `links` was sent for a record of a generator rule. Nothing was written.**

```json
{
  "ok": false,
  "reason": "invalid",
  "error": "Only a manual record has a data and links a person can edit."
}
```
**403: The deployment refuses the write, or the register could not be written. Nothing was written.**

```json
{
  "ok": false,
  "reason": "forbidden",
  "error": "Writing records is not permitted."
}
```
**404: No record has that id in that model.**

```json
{
  "ok": false,
  "reason": "not-found",
  "error": "record not found: example-rules/cross-zone-relation:example-grid/relationId=F9,zoneId=Z9"
}
```
