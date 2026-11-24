Sightline Prism keeps one current, auditable record of every asset in an estate. It collects asset data from the systems that already hold it. It matches the records that describe the same asset, and merges them into a consolidated dataset. Every value in that dataset records the source that supplied it and the reason it won.

## Challenges with Asset Inventories

A complete asset inventory is difficult to build and more difficult to keep current. An inventory of operational technology (OT) assets is the most difficult case.

### Limits of a CMDB-Based Inventory

- **Narrow discovery.** A CMDB usually collects from traditional IT sources: Windows and Linux hosts, virtual hosts and cloud platforms. Assets outside that reach stay out of the inventory, or someone enters them by hand.
- **No provenance and no merge.** A CMDB stores a value without the source and the reason behind it. Automatic collection requires deciding which source overwrites which, and accepting what each choice loses.
- **No write-back.** A CMDB does not send what it knows back to the source systems. An endpoint security (EDR) platform never receives the classification or criticality that the CMDB holds.

### Constraints on an OT Asset Inventory

- **The best data is in niche systems.** The richest OT asset data is often in specialised databases. An off-the-shelf connector for them is unlikely to exist.
- **Sites are isolated.** Many OT networks are air-gapped or offline. Their assets still belong in the central inventory.
- **Spreadsheets stay.** OT teams often keep asset data in spreadsheets, and they continue to maintain them. The inventory must read each new version, not import the spreadsheet once.

## Prism Capabilities

- **Merges by rule.** A sync rule sets a priority and a mode for each field from each source. The sync engine matches the records that describe one asset, and keeps the value from the source with the highest priority for each field.
- **Uses a common information model.** Every connector normalises its records to one set of Prism schemas. A field that no schema holds goes into an `extendedData` object, so a new source needs no schema change.
- **Keeps connectors simple.** A connector is any program that reads a source and sends a snapshot. Prism ships connectors in TypeScript, Python and PowerShell, and accepts snapshots from a connector written in any other language.
- **Works offline.** A connector at an isolated site writes its snapshot to a file. An operator uploads the file later, and the next sync merges it like any other snapshot.
- **Writes back.** Reverse rules send consolidated values to a target CMDB. They can also enrich a source system whose connector supports write-back, for example with asset criticality.
- **Runs hosted or on-premises.** Prism runs hosted in a Windmill workspace, or on-premises in self-managed containers.
- **Suits AI-assisted authoring.** Connectors are ordinary programs and rules are JSON files, so an AI assistant can draft both. The engine that applies them is deterministic. An operator approves each new asset, decommission or field change that a rule marks for review.

## Asset Data Flow Through Prism

![Asset data moves from the sources through connectors into connected datasets. The sync engine merges it into the consolidated dataset, directly or after operator review. Reverse rules then write back to a target CMDB and to the sources.](./img/prism-overview-light.svg#light)
![Asset data moves from the sources through connectors into connected datasets. The sync engine merges it into the consolidated dataset, directly or after operator review. Reverse rules then write back to a target CMDB and to the sources.](./img/prism-overview-dark.svg#dark)

| Stage | What happens |
|---|---|
| Collect | A connector reads one source, normalises each record to a Prism schema, and sends a snapshot. An offline connector writes the snapshot to a file for an operator to upload. |
| Match | The sync engine decides which records from different sources describe the same asset. |
| Merge | The engine compares each field with the value it already holds. Where the values differ, the source with the higher priority for that field wins. |
| Review | A change that a rule marks for review waits for an operator to approve, reject or decline it. |
| Write back | A reverse rule queues a consolidated value in an outbox. The connector at the target system, a CMDB or a source, makes the write. |

## Provenance and Review Guarantees

- **Provenance on every field.** Each value in the consolidated dataset records its source, that source's priority and the time it changed.
- **No silent new assets.** By default, a record that matches no existing asset waits for an operator to approve it.
- **Sources are read during a sync.** A sync reads stored snapshots. Only a write-back writes to a source, and the connector at that source performs it.
- **Snapshots are kept unchanged.** Each collection stays available for audit after later collections arrive.

## Ways to Run Prism

Prism is one product with two hostings. Standalone Prism runs on self-managed machines, and an operator uses the operator CLI in a terminal. Windmill-hosted Prism runs in a Windmill workspace, and an operator uses the review app in a browser. Both run the same engine, connectors and rules. Read the [Deployment options](/docs/prism/architecture/deployment-options) page for how to choose between the two hostings.

## Prism's Role in Sightline

Sightline's [Tetra, Lamina and Metron](https://docs.corvis.au/docs/sightline/architecture/overview) models describe an architecture, its risks and its compliance. Prism is the asset inventory part of Sightline. It keeps a current, auditable record of the assets in that estate, collected from the systems that hold them.
