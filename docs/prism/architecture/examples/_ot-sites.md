This example is an operational technology (OT) estate across two sites. Site A connects to the corporate network through a DMZ. Site B is air-gapped. Standalone Prism builds one inventory for both sites and keeps a corporate CMDB current. The organisation, its systems and every setting on this page are illustrative.

![Site A has a collector node that sends snapshots outbound to the Prism store in the corporate network. Site B has an offline connector that writes a snapshot file, which is carried out and uploaded. The central node runs the sync engine, and write-back updates the target CMDB.](./img/ot-sites-light.svg#light)
![Site A has a collector node that sends snapshots outbound to the Prism store in the corporate network. Site B has an offline connector that writes a snapshot file, which is carried out and uploaded. The central node runs the sync engine, and write-back updates the target CMDB.](./img/ot-sites-dark.svg#dark)

## The Estate

| Source | Where | What it knows well |
|---|---|---|
| OT asset database | Both sites | Controllers and field devices: model, firmware version, network zone |
| Site domain | Site A | Engineering workstations and servers: host name, operating system |
| Site register | Both sites | A spreadsheet that the site team keeps: location, owner, maintenance contact |

The OT asset database is a specialised product, and Prism ships no connector for it. This design adds one. A connector is any program that reads a source and sends a snapshot, so the site team writes it in Python against the database's export. [Writing a connector](/docs/prism/architecture/writing-a-connector) gives the method.

The site registers stay in use. Each site team keeps editing its own spreadsheet, and Prism reads each saved version.

## Component Placement Across the OT Sites

| Part | Runs on | Reason |
|---|---|---|
| Collector node for site A | A machine in the site A DMZ, installed with the `collector` role | It can reach the site sources and the Prism store. Nothing in the corporate network connects into the site. |
| Offline connector for site B | A machine inside site B | Nothing outside the site can reach its sources, and nothing inside can reach the store. |
| Prism store | PostgreSQL, in the corporate network | Both the collector node and the central node reach it. |
| Central node | A machine in the corporate network, installed with the `central` role | It runs the sync engine, and operators review changes in the review TUI. |

The collector node sends its snapshots outbound to the store.

## Collecting from the Air-Gapped Site

Site B never connects to the store. Its collection moves as a file.

1. The offline connector reads the site sources and writes a snapshot file.
2. A site operator copies the file to approved removable media, under the site's own transfer procedure.
3. An operator in the corporate network sends the file to the store. A connector can replay its own snapshot file, as the shipped CrowdStrike and Active Directory connectors do. Otherwise the operator uploads the records with the operator CLI.
4. The next sync merges the snapshot like any other.

The snapshot stays unchanged in the store after the upload. An auditor can compare it with the file that left the site.

## Rules for OT Sources

| Field | Wins | Why |
|---|---|---|
| Model, firmware version, network zone | OT asset database | It is the system that engineers maintain for OT devices. |
| Host name, operating system | Site domain | It reports what each Windows machine says about itself. |
| Location, owner, maintenance contact | Site register | Only the site team records them. |

Three settings matter more here than in an IT estate:

- **New assets wait for review.** Every rule keeps `newAssetMode` at `review`. A new device on an OT network is an event that someone must see.
- **Site B's grace period covers the transfer interval.** If site B sends a file once a month, its rules set `stalenessWindowMs` to more than a month. Its values then keep their priority between files.
- **The presence guard stays on.** A partial export from a site must not mark half its devices as decommissioned. With the default guard, a run in which more than half the present assets vanish makes no presence changes.

[Designing sync rules](/docs/prism/methodology/designing-rules) explains each setting.

## Write-Back

A reverse rule maps the consolidated dataset to the corporate CMDB. The CMDB gets one inventory of both sites, with the firmware, zone and owner of every device. The rule's mode is `review`, so an operator approves each change before it is queued.

The write needs a connector for the CMDB that supplies its own writer, as the Jira Assets connector does. Until one does, an operator completes the queued writes by hand. [Run a write-back](/docs/prism/usage/write-back) describes both paths.

This design writes nothing into the OT sites. A write into an OT network changes a production system. If a site wants enrichment, such as a criticality value in its asset database, keep the reverse rule in `review` mode. For site B, the queued writes cross the air gap by the same procedure as the snapshot file. A site operator then completes them by hand.
