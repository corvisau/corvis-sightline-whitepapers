This example is a corporate IT estate with cloud workloads, managed endpoints and an existing CMDB. Windmill-hosted Prism consolidates five sources, keeps a Jira Assets CMDB current, and tags each CrowdStrike host with its asset criticality. The organisation, its systems and every setting on this page are illustrative.

![Five sources feed Prism. Three are collected by scripts in the Windmill workspace and two by connectors on a Windows host. All of them write to an Atlas-hosted MongoDB store. The sync engine and review app work from the store. Write-back scripts update Jira Assets and tag each CrowdStrike host with its criticality.](./img/it-estate-light.svg#light)
![Five sources feed Prism. Three are collected by scripts in the Windmill workspace and two by connectors on a Windows host. All of them write to an Atlas-hosted MongoDB store. The sync engine and review app work from the store. Write-back scripts update Jira Assets and tag each CrowdStrike host with its criticality.](./img/it-estate-dark.svg#dark)

*Figure 1. Example architecture 1: Windmill-hosted Prism collects from five IT sources, and writes back to Jira Assets and CrowdStrike.*

## The Estate

| Source | What it knows well | Connector |
|---|---|---|
| Azure VMs | Cloud virtual machines, their size and their region | Azure VMs |
| CrowdStrike | Every managed endpoint: host name, operating system, prevention policy | CrowdStrike |
| Jira Assets | The CMDB: owner, business service and criticality | Jira Assets |
| Active Directory | Computer objects in the domain | Active Directory, in PowerShell |
| Asset register | A spreadsheet that the infrastructure team keeps: rack location and physical owner | Spreadsheet |

Jira Assets is both a source and the target. Prism reads the owner and criticality that people maintain in it, and writes back the values that other sources know better.

## Component Placement in the IT Estate

| Part | Runs on | Reason |
|---|---|---|
| Collector scripts for Azure VMs, CrowdStrike and Jira Assets | The Windmill workspace, on a schedule | Each one reads a cloud API that the workspace can reach. |
| Active Directory and spreadsheet connectors | A domain-joined Windows host, from a scheduled task | The directory and the file share are reachable only from inside the domain. |
| Prism store | Atlas-hosted MongoDB | The PowerShell Active Directory connector sends its snapshots to MongoDB. |
| Sync engine and review app | The Windmill workspace | Operators review changes in a browser. |
| Jira Assets write-back script | The Windmill workspace | The Jira Assets connector supplies its own writer. |
| CrowdStrike write-back script | The Windmill workspace | Prism ships no CrowdStrike writer, so this design adds one. |

Windmill-hosted and standalone parts mix freely, because every part reads and writes the same store. [Deployment options](/docs/prism/architecture/deployment-options) explains how to choose between the two hostings.

## Field Priorities Across the Sync Rules

Each source has one sync rule into the consolidated dataset. Each rule gives each field a priority, and the highest priority wins. This table shows the source that wins each field.

| Field | Wins | Why |
|---|---|---|
| Host name, operating system | CrowdStrike | The agent reports what the machine says about itself, every few minutes. |
| Cloud size and region | Azure VMs | Only the cloud platform knows them. |
| Owner, business service, criticality | Jira Assets | People maintain them in the CMDB on purpose. |
| Rack location | Asset register | Only the infrastructure team records it. |
| Domain membership | Active Directory | The directory is the authority for it. |

Criticality and owner are in `review` mode on every rule except the Jira Assets rule. A change to either from another source therefore waits for an operator. A new asset waits for an operator too, which is the default.

[Designing sync rules](/docs/prism/methodology/designing-rules) explains why each field's priority was chosen this way.

## Write-Back

One reverse rule maps the consolidated dataset to Jira Assets. It sends the host name, operating system and rack location, which the CMDB holds but other sources know better. Its mirror setting is in `review` mode, so a server that the CMDB does not hold yet is created only after an operator approves it.

The Jira Assets write-back script drains that outbox on a schedule. It writes each value to the matching Jira issue, and records the issue key of each record it creates.

A second reverse rule sends each asset's criticality from the consolidated dataset to CrowdStrike, as a host tag. The security team then sees the business importance of each endpoint in the Falcon console, beside its detections.

Prism ships no CrowdStrike writer, so this design adds a CrowdStrike write-back script. A write-back script is an ordinary program: it drains the outbox, writes each value, and records the result. [Writing a connector](/docs/prism/architecture/writing-a-connector) gives the method, and [Run a write-back](/docs/prism/usage/write-back) covers completing a write-back by hand while the script is in development.

## Operator Tasks in the IT Estate

- **Each day,** an operator opens the review app and decides the pending changes: new assets, owner changes and criticality changes.
- **Each week,** the infrastructure team saves the asset register. The spreadsheet rule has a staleness window longer than a week, so the register's rack locations keep their priority between saves.
- **When the CMDB disagrees,** the operator reads the provenance of the field in the consolidated dataset. It names the source, its priority and when the value changed.
