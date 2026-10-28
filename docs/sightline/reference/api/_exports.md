## `POST /exports`

Starts an export job and answers at once with its id. The body is a JSON object. `kind` names the export and the other fields are that kind's parameters. The kinds `metron-assessment-csv` and `metron-assessment-json` take `binding` (the workspace-relative path of an assessment file). The kinds `metron-portfolio-csv` and `metron-portfolio-json` take `package` (the workspace-relative path of a Metron package manifest). The report kinds are `lamina-report`, `tetra-report` and `metron-report`, and a server offers a report kind only when it has the report generators installed. Each takes `manifest` and `reportId`. The `manifest` is the workspace-relative path of a Lamina, Tetra or Metron manifest, or for `metron-report` an assessment file. The `reportId` is the id of a report that the manifest lists. Each also takes two optional fields. `inputs` is an object of string values for the inputs that the report declares. `riskId` reports one Risk of a Lamina model, from the causes, events and outcomes on its paths and on the paths of the Risks it holds; without it the report uses its own `risk`, or the whole model. A request with `dataSliceId` fails. A Lamina report also takes `suppliedModel` and `filterLabel`. `suppliedModel` is an object with a `bowtie` field that holds the model the screen shows. `filterLabel` is the text of its "Filter applied" line. The file is a DOCX, PPTX or XLSX, as the report's generator writes it. Without `suppliedModel` a report is built from the files on the server, so it carries no filter from a screen. The server runs the built-in report generators only. Poll `GET /exports/{id}` until `state` is `done`, then download the file. A missing or invalid parameter fails the job and the status carries the reason in `error`. A server that trusts its workspace, such as the VS Code extension host, also offers `lamina-report-batch`. It takes `manifest` and `reportId`. It runs the report once for each Risk of a Lamina model, twice (as Brief and Full) when the report declares a `printDetail` input, and fails for a model that defines no Risks. It writes each file into the model's `outputs/` folder. The file that the job returns is JSON. Its `outputDir` is the batch folder and its `files` list each file's `variant` and `path`, with paths relative to the workspace, and the `riskId` and `riskName` of its Risk. Other servers answer 400 for this kind. The PNG kinds are `lamina-png` and `tetra-png`, and a server offers a PNG kind only when PNG export is available on it. Each takes `manifest` (the workspace-relative path of a Lamina or Tetra manifest). `tetra-png` also takes an optional `sliceId` that names one data slice. `lamina-png` takes an optional `riskId` that names one Risk, which it draws as one aggregated event with its causes and outcomes, and rejects `sliceId`; `tetra-png` rejects `riskId`. Without `sliceId` (Tetra) or `riskId` (Lamina) the job renders the whole model. One job renders one image. Each also takes these optional fields. `size` is an object with `width` and `height` in pixels, each at most 8000. `scale` is a number above 0 and at most 4, for the device pixel ratio. `physicalCm` is an object with `width` and `height` in centimetres, each at most 1000, which the PNG records as its print size. `ratio` letterboxes the image. For `lamina-png`, `ratio` is `true`. For `tetra-png`, `ratio` is an object with `width` and `height`. A PNG job renders the files on the server, so it shows no unsaved edit. A PNG job that finds nothing to draw fails with the reason in `error`. A PNG job can stay `running` while it waits for its turn. Cancelling it frees its place. A server that does not offer exports answers 404 on every `/exports` route.

### Responses

**202: The job was accepted. It is `queued` until a worker is free, then `running`.**

```json
{
  "id": "00000000-0000-4000-8000-000000000000",
  "state": "queued"
}
```
**400: The body is not a JSON object, `kind` is missing, or no export has that kind.**

```json
{
  "error": "unknown export kind: example-kind"
}
```
**403: The server's read check refused the request.**

```json
{
  "error": "Reading Metron data is not permitted."
}
```

## `GET /exports/{id}`

Returns the job's `state`, its `progress` and, once it has finished, the file name and media type. `state` is one of `queued`, `running`, `done`, `failed` and `cancelled`. `progress` is `null` until the export reports its first step, then holds `done`, `total` and a `label`. A failed job carries `error`. A finished job stays available for about 15 minutes, and the server keeps at most 50 finished jobs, so an older id answers 404.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string | The export id that `POST /exports` returned. |

### Responses

**200: The job's current status.**

```json
{
  "id": "00000000-0000-4000-8000-000000000000",
  "kind": "metron-portfolio-csv",
  "state": "running",
  "progress": {
    "done": 1,
    "total": 2,
    "label": "resolving portfolio"
  }
}
```
**404: No export has this id, or it expired.**

```json
{
  "error": "export not found: 00000000-0000-4000-8000-000000000000"
}
```

## `DELETE /exports/{id}`

Cancels a queued or running job. A queued job never starts, and a running job is told to stop at its next step. A job that has already finished is returned unchanged.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string | The export id that `POST /exports` returned. |

### Responses

**200: The job's status after the cancel.**

```json
{
  "id": "00000000-0000-4000-8000-000000000000",
  "kind": "metron-portfolio-csv",
  "state": "cancelled",
  "progress": null
}
```
**404: No export has this id, or it expired.**

```json
{
  "error": "export not found: 00000000-0000-4000-8000-000000000000"
}
```

## `GET /exports/{id}/file`

Returns the finished file as an attachment, with the media type and file name that the status reports. The read check that guards `POST /exports` also guards this route.

| Parameter | In | Required | Type | Description |
|---|---|---|---|---|
| `id` | path | yes | string | The export id that `POST /exports` returned. |

### Responses

**200: The file. A CSV export answers `text/csv` and a JSON export answers `application/json`. The example shows the first columns of a portfolio CSV only.**

```csv
model_id,model,assessor,package_id,package_version,requirement_id
m1,grid,assessor@example.test,pkg-1,1,R-0001
```
**403: The server's read check refused the request.**

```json
{
  "error": "Reading Metron data is not permitted."
}
```
**404: No export has this id, or it expired.**

```json
{
  "error": "export not found: 00000000-0000-4000-8000-000000000000"
}
```
**409: The job has not finished, or it failed or was cancelled. `state` says which.**

```json
{
  "error": "export is running, not done",
  "state": "running"
}
```
