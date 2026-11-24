Sightline Prism is one product with two hostings. The hostings differ in where Prism runs and which interface the operator uses. They do not differ in what Prism does.

| Hosting | Where it runs | What the operator uses |
|---|---|---|
| Standalone Prism | Your own machines, as one or more nodes | The operator CLI, in a terminal |
| Windmill-hosted Prism | A Windmill workspace | The review app, in a browser |

Both hostings run the same engine, connectors and rules. A decision that an operator records in one has the same effect in the other.

## Two Kinds of Work, in Two Places

Prism splits its work by what each part must reach.

| Work | Where it runs | Why |
|---|---|---|
| Collect from a source | Near the source | Only a machine that can reach the source can read it. |
| Drain an outbox, which performs write-backs | Near the source | The write needs the same connectivity and credentials as the collection. |
| Match, merge and review | Centrally | The engine needs every connected dataset and the consolidated dataset. |

The two kinds of work meet in the Prism store. A collector writes its snapshot to the store. The engine reads the snapshot from the store and writes its write-backs to the outbox in the store.

## Standalone Prism

A standalone deployment is usually more than one node. The installer's `--role` flag chooses what each node does.

| Role | What the node does |
|---|---|
| `collector` | Collects from the sources it can reach |
| `central` | Runs the sync engine, the review TUI and the other operator CLI commands |
| `both` | Does both, on one machine |

The drain command, `prism-drain`, is part of the operator CLI. Run it on a machine that can reach the source it writes to.

```mermaid
flowchart TB
    subgraph SITE1["Site network"]
        SA[("Source system")] <--> N1["Collector node"]
    end
    subgraph CLOUD["Cloud network"]
        SB[("Source system")] <--> N2["Collector node"]
    end
    STORE[("Prism store<br/>MongoDB or PostgreSQL")]
    subgraph CORE["Central node"]
        direction LR
        ENG["Sync engine"]
        TUI["Review TUI"]
    end
    N1 <--> STORE
    N2 <--> STORE
    STORE <--> CORE
    classDef src fill:#e3eefb,stroke:#3d6fa8,color:#13263d
    classDef proc fill:#dff3f0,stroke:#1f8a80,color:#0d2e2b
    classDef store fill:#fbf0dc,stroke:#b07d1f,color:#3a2806
    classDef person fill:#f7e4ef,stroke:#a2477f,color:#3a1029
    classDef muted fill:#eceff3,stroke:#7b8594,color:#1f2630
    class SA,SB src
    class N1,N2,ENG,TUI proc
    class STORE store
```

Each node is a [runtime instance](/docs/prism/install/runtime-instance): a directory that holds that node's environment file, connector manifests and rules. Operational commands run from the runtime instance.

A single machine with the `both` role is enough for a trial. The local JSON store suits that case, because it needs no database server.

## Windmill-Hosted Prism

Windmill-hosted Prism deploys the collectors as scheduled Windmill scripts, and serves the review app in a browser.

```mermaid
flowchart LR
    subgraph WS["Windmill workspace"]
        direction TB
        COL["Collector scripts<br/>on a schedule"]
        APP["Review app"]
    end
    SRC[("Source systems")]
    STORE[("Prism store<br/>MongoDB or PostgreSQL")]
    CLI["Operator CLI<br/>optional"]
    SRC <--> COL
    COL <--> STORE
    APP <--> STORE
    CLI <--> STORE
    classDef src fill:#e3eefb,stroke:#3d6fa8,color:#13263d
    classDef proc fill:#dff3f0,stroke:#1f8a80,color:#0d2e2b
    classDef store fill:#fbf0dc,stroke:#b07d1f,color:#3a2806
    classDef person fill:#f7e4ef,stroke:#a2477f,color:#3a1029
    classDef muted fill:#eceff3,stroke:#7b8594,color:#1f2630
    class SRC src
    class COL,APP,CLI proc
    class STORE store
```

The review app records the same decisions as the review TUI. Both also cover a source that no connector can reach:

- A person uploads data for a source whose connector cannot run.
- A person completes a queued write-back by hand.

[The review app](/docs/prism/usage/review-app) describes both in the browser.

## The Hostings Are Not Exclusive

The operator CLI reads and writes an ordinary Prism store, and a Windmill-hosted store is an ordinary Prism store. An operator can run a CLI command against a Windmill-hosted deployment from a laptop, and the workspace shows the result.

Choose a hosting for where your operators work.

## Choose a Storage Backend

Prism stores its data in one of three backends.

| Backend | Use it for |
|---|---|
| MongoDB | Any deployment, and either hosting |
| PostgreSQL | Any deployment, and either hosting |
| Local JSON store | One operator on one machine. Windmill-hosted Prism does not support it. |

The backends do not share data. Choose the backend before you collect data that you want to keep. [Storage backends](/docs/prism/install/storage-backends) gives the settings for each one.

## Release Archive Contents

Corvis supplies each hosting as one archive. The archive holds compiled code for the engine, the connectors and the interface. An install needs no source checkout, and it reaches no package registry.
