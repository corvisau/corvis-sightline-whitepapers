On extension activation, Lamina, Tetra and Metron each send a single org-level activation ping. The ping supports commercial-licence enforcement and is disclosed in the telemetry section of the licence terms.

## What is sent

- Organisation identity: the Windows Active Directory DNS domain (`USERDNSDOMAIN`) when available. Otherwise the ping carries the domain portion only of your git `user.email`. The app discards the local part immediately and never reads it onto the payload.
- Commercial licence status: `absent`, `invalid`, `expired` or `valid`. When a licence is present, the ping also carries `licensedTo`, `licenseId`, `expiresAt` and `features`.
- Non-PII host metadata: a VS Code-assigned installation id (`machineId`), the session id, the VS Code version and the display language.
- The name of the extension (`lamina`, `tetra` or `metron`) and its running version.

The ping never carries your full email address, `USERDOMAIN` (NetBIOS), hostname or username.

## Cadence and behaviour

The ping fires once per activation and stores nothing on the machine. No UI element shows it, and a failed ping does not affect the app.

Activation also runs a licence-revocation check, described under Revocation in [Licensing](/docs/sightline/usage/shared/licensing).

## Licence status per app

The licence status in the ping is the status of the licence in force for the app that sends it. The licence is one file shared by every Sightline app on the machine, as [Licensing](/docs/sightline/usage/shared/licensing) describes.
