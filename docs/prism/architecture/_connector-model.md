A connector reads one type of source and normalises what it finds into a Prism schema. This document describes the connector model and the manifest that turns connector code into a running instance.

## Code Against Instance

The distinction is the centre of the model, and the current documents blur it.

| | What it is | Where it lives |
|---|---|---|
| **Connector** | The code that knows how to read one type of source. It is a library. | `packages/connectors/<type>/` in the code checkout |
| **Connector manifest** | One configured instance of that connector: which system, which credentials, which schema. | `.connectors/<type>.<instance>/manifest.json` in the runtime instance |

**One connector is driven by many manifests.** A CrowdStrike connector that serves three tenants needs three manifests, one per tenant, not three copies of the connector.

The **collector** is the runnable entry point of a connector: the thing an operator starts to perform a collection. The connector is the component; the collector is what runs.

```mermaid
flowchart LR
    subgraph checkout["Code checkout"]
        C["Connector<br/>packages/connectors/crowdstrike/"]
    end
    subgraph instance["Runtime instance"]
        M1["manifest<br/>crowdstrike.eu"]
        M2["manifest<br/>crowdstrike.us"]
    end
    C --> M1
    C --> M2
    M1 --> T1["table: crowdstrike.eu"]
    M2 --> T2["table: crowdstrike.us"]
```

## The Manifest

A manifest has six fields.

| Field | What it holds |
|---|---|
| `connectorType` | Which connector runs this instance |
| `instance` | The label that distinguishes this instance from others of the same type |
| `schema` | The asset schema this instance produces, such as `AssetComputer` |
| `credentials` | A map of logical name to credential reference |
| `config` | Settings for this instance that are not secret |
| `description` | Optional free text |

### The Naming Rule

`connectorType` and `instance` accept letters, digits and hyphens. They reject dots.

The reason is the identifier they combine into. A manifest feeds a table named `connectorType.instance` (`crowdstrike.eu` in the diagram above). A dot inside either segment would make that identifier ambiguous, so Prism refuses it at validation rather than producing a table nobody can address.

### Credentials Are References, Never Values

**A manifest never holds a credential.** Each entry in `credentials` maps a logical name to an environment variable reference, in the form `env:VARIABLE_NAME`.

Prism validates this. An entry that is not an `env:` reference is rejected when the manifest loads, not merely warned about.

This is a designed guarantee, and it has a practical consequence: a manifest is safe to commit to version control. It records which variable supplies each credential, and nothing more. The values live in the runtime instance's environment file, which is not committed.

Name the variables in any document or discussion of a connector. Never write a value, and never write a placeholder shaped like a real one.

## From Collection to a Table

A collector runs, reads its source, and produces a snapshot.

1. The collector loads its manifest and the environment for that instance.
2. It reads the source system, using the credentials the manifest references.
3. It normalises each record to the schema the manifest names.
4. It validates each record against that schema.
5. It builds a snapshot and transmits it to the instance's inbox.

**The first transmit registers the table.** An operator does not create a table by hand before a connector can use it. The connector's first successful transmit self-registers `connectorType.instance` in the table registry, and the sync engine can then target it.

That registration is also why a rule pointing at a table no source has ever transmitted to fails with an unknown-table error.

## The Boundary

A connector collects, normalises and transmits. Every judgement about what a value means belongs to the engine: which record it describes, which source outranks which, and what reaches the consolidated dataset. A connector never writes there.

Write-back is the one exception, and a narrow one. A connector drains the outbox and performs the write against its own source, because it holds the connectivity and the credentials for it. The engine still decides what to write. See [Sync engine](/docs/prism/architecture/sync-engine) for how it decides.

This separation is what makes a connector a public extension surface. It builds against the connector SDK alone.
