Sightline is source-available software under the Sightline Licence. It is not open source as the Open Source Initiative defines the term, because it limits who may use it and for what purpose.

The binding terms are in the `LICENSE` file that ships with the product. The summary at the top of that file is not the licence.

## How the apps enforce the licence

- **A commercial licence names the apps that it unlocks.** The licence carries one grant for each app: `edit:lamina`, `edit:tetra` and `edit:metron`.
- **Reading Metron data from Tetra needs its own grant.** The `view:metron` grant covers the findings badges and the coverage overlay in Tetra. It does not unlock editing.
- **Editing is licensed for each app.** A licence for one app leaves the other apps able to view and to export, but not to save.
- **One licence covers every Sightline app on a machine.** The apps store the licence once and share it.
- **An app with a missing, invalid or expired licence does not save.** Viewing and export stay usable.

See [Licensing](/docs/sightline/usage/shared/licensing) for the banner, the licence badge and the behaviour of each app.

## Deployment options

The licence applies in the same way to each of the three deployment options.

- **The same grants apply.** Local Community Sightline, Self-Hosted Sightline and Corvis Sightline each check the per-app grants described above.
- **A hosted deployment applies the grants to each save.** The operator provides one licence. A save by a signed-in user needs the grant for that app.
- **Documentation only means no support.** Self-Hosted Sightline carries no support obligation.

See [Deployment options](/docs/sightline/architecture/deployment-options).

## Telemetry

Each app sends one activation ping when it starts. The telemetry section of the licence terms discloses it. See [Telemetry](/docs/sightline/usage/shared/telemetry).
