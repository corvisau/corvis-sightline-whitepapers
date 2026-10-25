A Tetra model holds eight kinds of entity. Five are nodes that sit in the containment tree: zones, groups, devices, networks and information. Channels and flows connect nodes. A system collects nodes outside the tree. Each kind has one job, and a model stays readable when each entity does only that job.

## The entity kinds

| Kind                                | What it represents                                                                                        | Holds members           |
| ----------------------------------- | --------------------------------------------------------------------------------------------------------- | ----------------------- |
| Zone                                | A set of nodes that share common security requirements and need a common security boundary, as IEC 62443 defines a zone | Yes                     |
| Group                               | A grouping that adjusts the layout, with no meaning to rules                                              | Yes                     |
| Device | A machine, appliance or piece of software that sends or receives data                                     | Yes, mainly information |
| Network                             | A network that devices attach to                                                                          | No                      |
| Information                         | A named item of data that a device or zone holds                                                          | No                      |
| Channel                             | A conduit that could carry information flows between two nodes                                            | No                      |
| Flow                                | Information that moves between two nodes                                                                  | No                      |
| System                              | A collection of devices, zones, groups and networks that work together                                    | Lists its members by id |

[Tetra channels and flows](/docs/sightline/methodology/tetra-channels-and-flows) covers the last two connecting kinds in detail.

## Zones and groups

A zone and a group both contain other nodes, and both draw a box around them. The difference is what the model does with the box.

A zone follows the IEC 62443 definition. It groups nodes that share common security requirements, and its boundary separates them from other nodes. A zone can follow an existing network security control, but it does not have to. A node belongs to its nearest enclosing zone, and a zone belongs to itself. When a channel or flow joins two nodes in different zones, the relationship is cross-zone. The [channels and flows page](/docs/sightline/methodology/tetra-channels-and-flows#channels-flows-and-zones) explains how rules use that fact.

A group adjusts the layout. It tidies the canvas, and it never changes how a rule reads the model.

A zone carries three optional fields that look alike and do different jobs:

| Field | Job |
|---|---|
| Zone type | Classifies the zone, for example `OT-L3` or `IT`, and sets the zone colour |
| SL-T | States the target security level, and shows as a red pill in the zone header |
| **Zone Trust** | Holds a display-only label, and shows as a grey pill. See [Tetra zone trust](/docs/sightline/usage/tetra/zone-trust) |

## Devices, networks and information

A device is the unit that does work. Give each machine its own device, and attach it to the networks it uses through its `connections`. A device that connects to a network without an authored channel receives a channel of type `network` with the protocol `ip` automatically.

A network represents the network itself, not a link. Use a network for a segment that several devices share, and a channel for a conduit between two specific nodes. A network can carry address ranges. See [Network Addressing](/docs/sightline/usage/tetra/network-addressing).

Information is a point, not a connection. It names a piece of data, such as a setpoint or a historian feed, that a device or zone holds. A flow can start or end at an information item, which lets the model say which data moves instead of only which devices talk.

## Systems

A system gathers devices, zones, groups and networks that serve one purpose. A device can belong to a system and still sit in a zone. A system draws no box, so it describes a function without changing the layout. A data slice can select system entities by kind. See [Tetra Data Slice Filtering](/docs/sightline/usage/tetra/data-slice-filtering).

## Choosing between them

| You want to | Use |
|---|---|
| Group nodes that share security requirements behind a common boundary | A zone |
| Adjust the layout of nodes on the canvas | A group |
| Represent one machine | A device |
| Represent a segment that devices share | A network |
| Record a conduit that could carry flows between two nodes | A channel |
| Record what data moves between two nodes | A flow |
| Name a piece of data a device holds | Information |
| Describe a function across zones | A system |

Repeated bodies belong in a [template](/docs/sightline/usage/tetra/templates), and group labels follow the rules in [Group labels](/docs/sightline/usage/tetra/group-labels).
