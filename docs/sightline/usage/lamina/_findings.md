Lamina lists the Metron findings and the Tetra records in a model's scope, and creates a cause that links one of them. It never writes a finding or a record. A cause stores the link, and Lamina shows the state of each linked finding or record on the cause.

## The Findings inbox

Click **Open Findings Inbox** in the toolbar. The inbox has two lists.

- **Metron findings.** The assessment findings of the Tetra model that the Lamina model names under `tetraModel`. When the manifest names none, Lamina lists the findings of every Tetra model in the workspace. A Metron finding needs no zone, so this list shows even when the manifest declares no `tetraZones`.
- **Tetra records.** The records of that Tetra model in the zones that the manifest declares under `tetraZones`. See [Record links](/docs/sightline/usage/lamina/finding-links).

Each list shows only the findings and records that no cause of the open model links, so the answer to unlinked is per model. A record that its rule closes does not appear. A finding appears in the Metron list only when its rule does not hide it.

An empty list says why: no Metron findings are in scope, every one has a cause, or the findings could not be read. An authored findings file is not read. See [Findings in Metron](/docs/sightline/usage/metron/findings).

## Creating a cause from a finding

Click **Create cause from finding** on a Metron finding, or **Create cause** on a Tetra record. The inbox closes and the props drawer opens on its new-cause surface. The cause is a draft, prefilled with the finding's title as its label and, for a Metron finding, the finding's description. Adjust any field, pick the destination file, and press **Create**.

The prefill carries the record link, pinned at the state of the finding now, and a snapshot of the label, description and tags that the finding supplies. The cause starts **Current** and approved. **Create** stages the cause with the link in the pool of unsaved changes, and the next **Save all** commits both together. Nothing is written to any other file. **Cancel** closes the drawer and leaves the finding in the inbox.

## The linked finding on the cause

A cause that links a finding shows the finding in its drawer, with its state: **Current**, **Data changed**, **Source changed**, **Data and source changed**, **Not accepted yet**, **Closed**, **Not found** or **Cannot be read now**. See [Record links](/docs/sightline/usage/lamina/finding-links) for **Accept**, **Withdraw approval**, **Remove link** and the explanation of what changed.

A finding link is stored once, in the cause's own record. A finding that a cause links is listed by Metron too, under the finding's usages. For how the apps share findings, see the [Cross-app workflows](/docs/sightline/architecture/cross-app-workflows) page.
