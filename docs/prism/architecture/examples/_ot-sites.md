This example is an operational technology (OT) estate whose network has four zones: SaaS, the IT network, an IT-OT DMZ and the OT network. Standalone Prism runs in containers in the IT network. It reads the IT and OT CMDBs, collects from three kinds of OT site, and keeps the OT CMDB current. The organisation, its systems and every setting on this page are illustrative.

![The estate spans four zones. In the SaaS zone, Prism reads ServiceNow, the IT CMDB, and Jira Assets, the OT CMDB, and writes back to Jira Assets. In the IT network, Prism runs as two containers: the Prism container and a PostgreSQL container. In the IT-OT DMZ, Prism polls Tenable OT Security, which polls the devices at site A, and an operator uploads site C's files from a file transfer server. Site B's collector node pushes snapshots to the store.](./img/ot-sites-light.svg#light)
![The estate spans four zones. In the SaaS zone, Prism reads ServiceNow, the IT CMDB, and Jira Assets, the OT CMDB, and writes back to Jira Assets. In the IT network, Prism runs as two containers: the Prism container and a PostgreSQL container. In the IT-OT DMZ, Prism polls Tenable OT Security, which polls the devices at site A, and an operator uploads site C's files from a file transfer server. Site B's collector node pushes snapshots to the store.](./img/ot-sites-dark.svg#dark)

*Figure 1. Example architecture 2: Standalone Prism in the IT network collects from three kinds of OT site, and writes back to the OT CMDB.*

## Network Zones

| Zone | What it holds | What crosses its boundary |
|---|---|---|
| SaaS | ServiceNow, the IT CMDB, and Jira Assets, the OT CMDB | The Prism container reads both over their APIs, and writes back to Jira Assets. |
| IT network | The Prism container and the PostgreSQL container | Connections out to the SaaS APIs and the DMZ, and snapshots in from the site B collector node |
| IT-OT DMZ | Tenable OT Security and a file transfer server | The Prism container polls the Tenable API, and an operator uploads files from the file transfer server. |
| OT network | Three sites, each collected a different way | Tenable polls site A, site B pushes snapshots out, and site C's files leave by hand. |

## Three Kinds of OT Site

- **Site A: polled by Tenable.** Tenable OT Security in the DMZ polls the site's controllers, HMIs, switches, firewalls and engineering workstations. Prism installs nothing in the site. A Tenable connector in the Prism container reads Tenable's inventory over its API.
- **Site B: collector node.** A collector node inside the site runs a connector for each local source: network threat detection, network management (NMS) and on-premises antivirus. It pushes each snapshot to the Prism store through one firewall rule.
- **Site C: offline.** Nothing outside the site can reach its sources, and nothing inside it can reach the store. A site operator exports the sources to files, and the files leave the site on approved removable media.

## The Estate

| Source | Zone or site | Collected by | What it knows well |
|---|---|---|---|
| ServiceNow | SaaS | ServiceNow connector | Assets that IT and OT share, such as historians and jump hosts: business service, IT owner |
| Jira Assets | SaaS | Jira Assets connector | The OT CMDB's own records: asset ID, criticality |
| Tenable OT Security | IT-OT DMZ | Tenable connector | Site A devices: model, firmware version, IP address |
| Network threat detection | Site B | Collector node | Every device that talks on the site network: MAC address, IP address |
| Network management (NMS) | Site B | Collector node | Switches, routers and firewalls: model, firmware version |
| On-premises antivirus | Site B | Collector node | Windows hosts: host name, operating system |
| OT Active Directory | Site C | Offline export | Workstations and servers in the OT domain: host name, operating system |
| OT asset spreadsheet | Site C | Offline export | Location, owner and maintenance contact, which only the site team records |
| PLC configuration software | Site C | Offline export | Controllers: model, firmware version, module configuration |

Prism ships connectors for Jira Assets, Active Directory and spreadsheets. This design adds the rest: ServiceNow, Tenable, the three site B sources and the PLC configuration software. A connector is any program that reads a source and sends a snapshot. [Writing a connector](/docs/prism/architecture/writing-a-connector) gives the method.

## Component Placement Across the Zones

| Part | Runs in | Reason |
|---|---|---|
| Prism container | An IT host | It runs the sync engine, the operator CLI, and the connectors for ServiceNow, Jira Assets and Tenable. Operators review changes in the review TUI. |
| PostgreSQL container | The same IT host | It holds the Prism store. The Prism container and the site B collector node reach it. |
| Tenable OT Security | The IT-OT DMZ | It polls site A, and the Prism container polls it. Nothing in the IT network connects into the site. |
| File transfer server | The IT-OT DMZ | Removable media from site C connects to this server, not to a machine in the IT network. |
| Collector node | Site B, installed with the `collector` role | It reaches the site sources. One firewall rule lets it reach the PostgreSQL container, and nothing else. |
| Offline export | Site C | The site has no network path out. |

## Collecting from the Offline Site

Site C never connects to the store. Its collection moves as files.

1. A site operator exports each source to a file. The Active Directory computers go out as a JSON document of records.
2. The operator copies the files to approved removable media, under the site's own transfer procedure.
3. An operator copies the files from the media to the file transfer server.
4. An operator in the IT network loads each file into Prism. The spreadsheet and PLC configuration connectors read their files. The review TUI uploads the Active Directory document into its source's inbox.
5. The next sync merges the snapshots like any other.

Each snapshot stays unchanged in the store after the upload. An auditor can compare it with the file that left the site.

## Rules for OT Sources

| Field | Wins | Why |
|---|---|---|
| Model, firmware version | PLC configuration software, then Tenable OT Security, then the NMS | The configuration software holds what engineers loaded. Tenable and the NMS read what each device reports. |
| MAC address, IP address | Network threat detection | It sees every device that talks on the site network. |
| Host name, operating system | OT Active Directory, then on-premises antivirus | Both report what each Windows machine says about itself. |
| Location, owner, maintenance contact | OT asset spreadsheet | Only the site team records them. |
| Asset ID, criticality | Jira Assets | The OT CMDB is the system of record for them. |
| Business service, IT owner | ServiceNow | IT records them for the assets that IT and OT share. |

Three settings matter more here than in an IT estate:

- **New assets wait for review.** Every rule keeps `newAssetMode` at `review`. A new device on an OT network is an event that someone must see.
- **Site C's grace period covers the transfer interval.** If site C sends files once a month, its rules set `stalenessWindowMs` to more than a month. Its values then keep their priority between transfers.
- **The presence guard stays on.** A partial export from a site must not mark half its devices as decommissioned. With the default guard, a run in which more than half the present assets vanish makes no presence changes.

[Designing sync rules](/docs/prism/methodology/designing-rules) explains each setting.

## Write-Back

One reverse rule maps the consolidated dataset to Jira Assets, the OT CMDB. Jira Assets gets one inventory of every site, with the firmware, location and owner of each device. The rule's mode is `review`, so an operator approves each change before it is queued. The Jira Assets write-back script in the Prism container drains the outbox.

Prism reads ServiceNow and never writes to it. The IT CMDB stays under the IT team's own change process.

This design writes nothing into the OT sites. A write into an OT network changes a production system.
