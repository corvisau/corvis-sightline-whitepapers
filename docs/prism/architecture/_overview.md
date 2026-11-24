Sightline Prism aims to solve the challenge of collecting, aggregating, reconciling and maintaining asset data that is spread across many systems. It collects asset data from the systems that already hold it. It matches the records that describe the same asset and merges them into one consolidated dataset. Every value in that dataset records the source that supplied it and the priority that let it win.

**Prism is not intended to be a CMDB. It is an integration layer that sits between a CMDB and the data sources.**

## Challenges with Asset Inventories

A complete asset inventory is difficult to build and more difficult to keep current. An inventory of operational technology (OT) assets is the most difficult case.

### Limits of a CMDB-Based Inventory

- **IT-centred discovery.** A CMDB's discovery and integrations usually centre on IT sources: Windows and Linux hosts, virtual hosts and cloud platforms. An asset outside that reach needs an integration built for it, or someone enters it by hand.
- **Limited provenance and merge.** Some CMDBs, such as ServiceNow, reconcile sources with precedence rules. Those rules cover only the sources that the CMDB integrates with. How much a value records about its source, and why it won, varies by product and configuration.
- **Limited write-back.** A CMDB rarely sends what it knows back to the source systems without custom integration work. An endpoint security (EDR) platform, for example, seldom receives the criticality that the CMDB holds.

### Constraints on an OT Asset Inventory

- **The best data is in niche systems.** The richest OT asset data is often in specialised databases. An off-the-shelf connector for them is unlikely to exist.
- **Sites are isolated.** Many OT networks are air-gapped or offline. Their assets still belong in the central inventory.
- **Spreadsheets stay.** OT teams often keep asset data in spreadsheets, and they continue to maintain them. The inventory must read each new version, not import the spreadsheet once.

## Prism Capabilities

- **Merges by rule.** A sync rule sets a priority and a mode for each field from each source. The sync engine matches the records that describe one asset, and keeps the value from the source with the highest priority for each field.
- **Uses a common information model.** Every connector normalises its records to one set of Prism schemas. A field that no schema holds goes into an `extendedData` object, so a source's extra fields need no schema change.
- **Keeps connectors simple.** A connector is any program that reads a source and sends a snapshot. Prism ships connectors in TypeScript, Python and PowerShell. A connector in another language can write its records to the store directly, or to a JSON document that an operator uploads.
- **Works offline.** At an isolated site, a connector's records can go to a file instead of the store. An operator uploads the file later, and the next sync merges it like any other snapshot.
- **Writes back.** Reverse rules queue consolidated values for a target CMDB, or for a source system such as an EDR platform. A connector with a writer performs each write. Prism ships a writer for Jira Assets, and an operator can complete other writes by hand.
- **Runs hosted or on-premises.** Prism runs hosted in a Windmill workspace, or on-premises on self-managed machines or site-built containers.
- **Suits AI-assisted authoring.** Connectors are ordinary programs and rules are JSON files, so an AI assistant can draft both. The engine that applies them is deterministic. An operator approves each new asset, decommission or field change that a rule marks for review.

## Scope Limits of Prism

- **Not a CMDB.** Prism does not replace a configuration management database such as Jira Assets or ServiceNow. Keep the CMDB as the system of record, and let Prism feed it through reverse rules.
- **Not a discovery tool.** Prism does not scan networks or probe devices. Its connectors read systems that already hold asset data, such as an EDR platform, a directory or a spreadsheet, or the machine they run on.

## Asset Data Flow Through Prism

![Asset data moves from the sources through connectors into connected datasets. The sync engine merges it into the consolidated dataset, directly or after operator review. Reverse rules then write back to a target CMDB and to the sources.](./img/prism-overview-light.svg#light)
![Asset data moves from the sources through connectors into connected datasets. The sync engine merges it into the consolidated dataset, directly or after operator review. Reverse rules then write back to a target CMDB and to the sources.](./img/prism-overview-dark.svg#dark)

*Figure 1. Asset data flows from the sources through Prism into the consolidated dataset, and back out to a target CMDB and the sources.*

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

## AI Capabilities in Prism

Prism contains no AI model, and no AI decides what reaches the consolidated dataset. The sync engine applies each rule deterministically, and an operator approves each change that a rule marks for review.

Prism gives an AI assistant defined structures to work with:

- **Defined schemas.** Every record conforms to a Prism schema, so an assistant reads a record without guessing its shape.
- **A read-only inspection API and MCP server.** An assistant that connects through the Model Context Protocol (MCP) can list the tables, read the latest snapshot from a source, and find an asset and its field history. It can also list pending changesets and deferrals, read the sync rules, and check the write-back outbox. No tool changes data.
- **Rules as JSON files.** An assistant can draft a sync rule for an operator to review before it runs.

An AI coding assistant also makes connector development faster. A connector is an ordinary program that reads one source and sends a snapshot. An assistant can draft one from the source's API documentation and an existing connector.

## Prism's Role in Sightline

Sightline's [Tetra, Lamina and Metron](https://docs.corvis.au/docs/sightline/architecture/overview) models describe an architecture, its risks and its compliance. Prism is the asset inventory part of Sightline. It keeps a current, auditable record of the assets in that estate, collected from the systems that hold them.
