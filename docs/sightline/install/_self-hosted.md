Self-Hosted Sightline runs Lamina, Tetra and Metron in a browser from one container on a host that you operate. The container serves the web shell and one storage server for all three apps. Self-Hosted Sightline has documentation only and no support. See [Deployment options](/docs/sightline/architecture/deployment-options).

This page is deployment and administration reference for the person who runs the container. Users work through the apps in the browser.

## Prerequisites

- A host with a container engine and a copy of the Sightline source
- A directory that holds your models, as described in [Prerequisites](/docs/sightline/install/prerequisites)
- A signed Sightline licence that names the apps you want to save in
- A reverse proxy that terminates TLS in front of the container

The container serves plain HTTP. It checks each request for a signed-in session or a bearer token. Do not publish its port beyond a trusted network without the proxy.

## Build the Image

From the root of the source copy, run the build. The build needs network access to fetch dependencies.

```bash
docker build -f deploy/hosted/Dockerfile -t example/sightline-hosted .
```

The default image has no PNG export. To add it, build with the `WITH_PNG` argument. This adds Chromium to the image and makes it several hundred megabytes larger.

```bash
docker build -f deploy/hosted/Dockerfile --build-arg WITH_PNG=true -t example/sightline-hosted .
```

## Prepare the Directories

The image holds no model, token, licence or signing key. You mount the directories below. The sign-in directory applies only when users sign in with a username and password.

| Container path | Holds | Access |
|---|---|---|
| `/data/models` | Every Tetra, Lamina and Metron model | Read and write |
| `/config` | `tokens.json` and `licence.json` | Read only |
| `/var/log/sightline` | The audit log, one JSON line for each save | Read and write |
| `/var/lib/sightline/auth` | The sign-in database | Read and write |

The container runs as user ID 10001. Give that user write access to the model, audit and sign-in directories on the host. Make `tokens.json` and `licence.json` readable by that user.

Model ids must be unique across the model directory, not only within one app. A Tetra model and a Lamina model with the same id answer 409.

### Choose How Users Sign In

The server accepts two kinds of credential, alone or together:

- A username and password. Users sign in on the shell, and the server keeps the session in a cookie.
- A bearer token from `tokens.json`. Scripts, CI and the `sightline-api` skill send these.

A deployment needs at least one of them.

### Set up Username and Password Sign-In

Three settings turn this on. Set all three or none.

| Variable | Meaning |
|---|---|
| `BETTER_AUTH_URL` | The public address of the deployment, such as `https://sightline.example.com` |
| `BETTER_AUTH_SECRET` | A random key of 32 characters or more. It signs the session cookie. Keep it out of the image and out of version control |
| `AUTH_DATABASE` | The path of the sign-in database, such as `/var/lib/sightline/auth/auth.sqlite`. The path must lie outside `MODEL_ROOT` |

A browser write must come from the address in `BETTER_AUTH_URL`. The server refuses a write from any other site with 403. Reach the deployment through that one address.

Create each user with the user command in the running container. The command asks for the password twice and does not echo it.

```bash
docker exec -it sightline node apps/web/dist/server/hostedUsers.cjs add ada --role editor --name "Ada Lovelace"
```

A password has at least 12 characters. A username has 3 to 30 letters, digits, underscores or dots, and the server ignores case. For a script, add `--password-stdin` and pipe the password in.

| Command | Effect |
|---|---|
| `add <username>` | Creates a user. The `--role` option is `reader` or `editor` and defaults to `reader` |
| `set-password <username>` | Replaces the password and signs the user out everywhere |
| `set-role <username> <role>` | Changes the role. It applies to the next request of the user |
| `remove <username>` | Deletes the user and the sessions of the user |
| `list` | Lists the users with their roles |

Users cannot register themselves, and the server sends no email. An administrator resets a forgotten password with `set-password`. The role rules in the next section apply to every user.

A user opens the address of the deployment and sees a form for the username and password. The form has a button that switches to an access token for a user who holds one. After sign-in, the toolbar shows the display name of the user. **Sign out** ends the session on the server, so the browser cannot reuse it. A deployment with tokens only shows the access token form and no name.

### Write the Token File

Write `tokens.json` when scripts need bearer tokens, or when you do not use username and password sign-in. With that sign-in on, the server ignores a missing `tokens.json`.

`tokens.json` maps each access token to an identity. The role is `reader` or `editor`. A reader opens and exports models. In the web shell, every control that edits a model is disabled for a reader, and so is **New**. An editor also saves and creates models.

```json
{
  "example-editor-token": { "userId": "editor-1", "orgId": "example-org", "role": "editor" },
  "example-reader-token": { "userId": "reader-1", "role": "reader" }
}
```

Use a long random value for each token. The server records the `userId` and `orgId` of the token in the audit log.

### Add the Licence

Save the signed licence as `licence.json`. The file holds either the licence string or the JSON that the VS Code extension stores. The server reads the file again on every check, so a replaced licence applies to the next save.

The licence and the role together decide who saves. A save needs an editor and a licence that names the app. Without a licence file, reading and export work and every save is refused. A tab shows Save disabled when the licence does not name its app, or when the caller is a reader, before you try to save. A replaced licence shows in an open tab after you reload it. See [Licensing](/docs/sightline/usage/shared/licensing).

## Start the Container

```bash
docker run -d --name sightline \
  -p 4960:4960 \
  -v "$PWD/models:/data/models" \
  -v "$PWD/config:/config:ro" \
  -v "$PWD/audit:/var/log/sightline" \
  -v sightline-auth:/var/lib/sightline/auth \
  -e BETTER_AUTH_URL=https://sightline.example.com \
  -e BETTER_AUTH_SECRET \
  -e AUTH_DATABASE=/var/lib/sightline/auth/auth.sqlite \
  example/sightline-hosted
```

The `-e BETTER_AUTH_SECRET` option passes the value from your shell. A deployment that uses tokens only leaves out the last four options.

The source copy includes a Compose example in `deploy/hosted/compose.example.yaml` with the same mounts. The container reports healthy when the shell answers.

Open the address of the proxy in a browser. Sign in with the username and password of a user, or with an editor token. The model list shows the models from every app.

## Install Sightline as an App

A browser can install the shell as an app. The app opens in its own window with the Sightline name and icon. The browser installs only an address that uses HTTPS, which the proxy in front of the container provides. The address `localhost` also works for a trial.

| Browser | How to install |
|---|---|
| Chrome and Edge, on desktop and Android | Select the install icon in the address bar, or choose the install option in the browser menu. While the browser offers the install, the toolbar also shows an **Install Sightline** button. |
| Safari on macOS | Choose **File > Add to Dock**. |
| Firefox 143 or later on Windows | Pin the site to the taskbar as a web app. The Microsoft Store build of Firefox does not offer this. |
| Firefox on macOS and Linux | The browser cannot install a site, so Sightline stays in a tab. |

## Settings

The image sets each variable below. Override one with the `-e` option of `docker run` when your layout differs.

| Variable | Image default | Meaning |
|---|---|---|
| `PORT` | `4960` | The port the server listens on inside the container |
| `MODEL_ROOT` | `/data/models` | The model directory |
| `AUTH_TOKENS_FILE` | `/config/tokens.json` | The token file. The server ignores a missing file when username and password sign-in is on |
| `BETTER_AUTH_URL` | none | The public address of the deployment. Turns on username and password sign-in with the next two |
| `BETTER_AUTH_SECRET` | none | The key that signs the session cookie |
| `AUTH_DATABASE` | none | The sign-in database. The path must lie outside `MODEL_ROOT` |
| `LICENCE_FILE` | `/config/licence.json` | The licence file |
| `AUDIT_LOG` | `/var/log/sightline/audit.jsonl` | The audit log. The path must lie outside `MODEL_ROOT` |
| `MAX_BROWSERS` | `1` | The Chromium instances at once for each app's PNG exports. It has no effect without `WITH_PNG` |

The image also sets the variables for report export and PNG export. Leave them unchanged.

## Back up and Update

Back up the model directory and the audit directory. With username and password sign-in, back up the sign-in database too, because it holds the users and their passwords. The signing key, token and licence files are your own copies.

A new `BETTER_AUTH_SECRET` signs every user out. The users and passwords stay.

To update, build a new image from the newer source copy. Stop the container, remove it and run the new image with the same mounts. Models stay on the host, so the update does not touch them. Read [Compatibility](/docs/sightline/upgrade/compatibility) before you move to a new version.

The container sends every page and bundle with `Cache-Control: no-cache` and an `ETag`. A browser checks for a new version on each load, so an update shows without a hard refresh. A CDN or reverse proxy in front of the container must honour these headers. A rule that sets a fixed cache lifetime keeps the old bundles after an update. Copies cached before this change carry no headers, so purge the cache of the CDN or proxy once when you first update to a version that includes it.
