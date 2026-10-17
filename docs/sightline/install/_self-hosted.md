Self-Hosted Sightline runs Lamina, Tetra and Metron in a browser from one container on a host that you operate. The container serves the web shell and one storage server for all three apps. Self-Hosted Sightline has documentation only and no support. See [Deployment options](/docs/sightline/architecture/deployment-options).

## Prerequisites

- A host with a container engine and a copy of the Sightline source
- A directory that holds your models, as described in [Prerequisites](/docs/sightline/install/prerequisites)
- A signed Sightline licence that names the apps you want to save in
- A reverse proxy that terminates TLS in front of the container

The container serves plain HTTP and checks one bearer token on each request. Do not publish its port beyond a trusted network without the proxy.

## Build the image

From the root of the source copy, run the build. The build needs network access to fetch dependencies.

```bash
docker build -f deploy/hosted/Dockerfile -t example/sightline-hosted .
```

The default image has no PNG export. To add it, build with the `WITH_PNG` argument. This adds Chromium to the image and makes it several hundred megabytes larger.

```bash
docker build -f deploy/hosted/Dockerfile --build-arg WITH_PNG=true -t example/sightline-hosted .
```

## Prepare the directories

The image holds no model, token or licence. You mount three directories.

| Container path | Holds | Access |
|---|---|---|
| `/data/models` | Every Tetra, Lamina and Metron model | Read and write |
| `/config` | `tokens.json` and `licence.json` | Read only |
| `/var/log/sightline` | The audit log, one JSON line for each save | Read and write |

The container runs as user ID 10001. Give that user write access to the model and audit directories on the host. Make `tokens.json` and `licence.json` readable by that user.

Model ids must be unique across the model directory, not only within one app. A Tetra model and a Lamina model with the same id answer 409.

### Write the token file

`tokens.json` maps each access token to an identity. The role is `reader` or `editor`. A reader opens and exports models. In the web shell, every control that edits a model is disabled for a reader, and so is **New**. An editor also saves and creates models.

```json
{
  "example-editor-token": { "userId": "editor-1", "orgId": "example-org", "role": "editor" },
  "example-reader-token": { "userId": "reader-1", "role": "reader" }
}
```

Use a long random value for each token. The server records the `userId` and `orgId` of the token in the audit log.

### Add the licence

Save the signed licence as `licence.json`. The file holds either the licence string or the JSON that the VS Code extension stores. The server reads the file again on every check, so a replaced licence applies to the next save.

The licence and the role together decide who saves. A save needs an editor token and a licence that names the app. Without a licence file, reading and export work and every save is refused. A tab shows Save disabled when the licence does not name its app, or when the token is a reader token, before you try to save. A replaced licence shows in an open tab after you reload it. See [Licensing](/docs/sightline/usage/shared/licensing).

## Start the container

```bash
docker run -d --name sightline \
  -p 4960:4960 \
  -v "$PWD/models:/data/models" \
  -v "$PWD/config:/config:ro" \
  -v "$PWD/audit:/var/log/sightline" \
  example/sightline-hosted
```

The source copy includes a Compose example in `deploy/hosted/compose.example.yaml` with the same mounts. The container reports healthy when the shell answers.

Open the address of the proxy in a browser and sign in with an editor token. The model list shows the models from every app.

## Settings

The image sets each variable below. Override one with the `-e` option of `docker run` when your layout differs.

| Variable | Image default | Meaning |
|---|---|---|
| `PORT` | `4960` | The port the server listens on inside the container |
| `MODEL_ROOT` | `/data/models` | The model directory |
| `AUTH_TOKENS_FILE` | `/config/tokens.json` | The token file |
| `LICENCE_FILE` | `/config/licence.json` | The licence file |
| `AUDIT_LOG` | `/var/log/sightline/audit.jsonl` | The audit log. The path must lie outside `MODEL_ROOT` |
| `MAX_BROWSERS` | `1` | The Chromium instances at once for each app's PNG exports. It has no effect without `WITH_PNG` |

The image also sets the variables for report export and PNG export. Leave them unchanged.

## Back up and update

Back up the model directory and the audit directory. The token and licence files are your own copies.

To update, build a new image from the newer source copy. Stop the container, remove it and run the new image with the same mounts. Models stay on the host, so the update does not touch them. Read [Compatibility](/docs/sightline/upgrade/compatibility) before you move to a new version.
