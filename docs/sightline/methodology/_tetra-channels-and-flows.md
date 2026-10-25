A channel and a flow both join two nodes, and they answer different questions. A channel is a conduit that could carry information flows between two nodes. A flow states that information moves between them. A model can hold a channel with no flow, a flow with no channel, or both between the same pair.

## Channel against flow

| | Channel | Flow |
|---|---|---|
| Represents | A conduit that could carry flows | Information that moves |
| Joins | Any two nodes | Information items, or devices when no information is named |
| The kind is set by | `channelType` | `type` |
| Line style shows | The channel type, or `lineStyle` | How the flow starts: `interactive`, `programmatic` or `offline` |
| Line colour is set by | The channel type, or `lineColor` | The flow type |

A channel carries other flows and holds no information of its own. An API that other flows could use as a general carrier is a channel. An API that does one job is a flow.

A flow does not name a channel, and a channel does not name a flow. A path step on a flow can borrow the route of a channel with `via`, and that only changes how the line is drawn.

Model the channel first, then add flows for the information that matters. A channel with no flows still tells a rule that information could move between two nodes. A flow with no channel records information movement that the model does not tie to a particular conduit.

## Channel types

A channel has a `channelType` that says what kind of conduit it is.

| Value | Shown as | Use it for |
|---|---|---|
| `network` | Network | A conduit over a network or a direct link. Most channels use this value, and it is the default |
| `virtual` | Virtual | A tunnel with endpoints of its own that rides other channels, such as an IPsec VPN between two site firewalls |
| `hosting` | Hosting | A node that runs on another, such as a virtual machine on its hypervisor or a VLAN on a switch. The host has full access to the guest's memory, storage and processing. The canvas draws it dotted |

The channel type does not describe the physical medium, because the medium changes along a route.

A connection that carries no information is not a channel, so the model has no type for a cable or a mechanical coupling.

A channel also has a `status` of `online` or `offline`.

## Protocol stack

A channel and a flow can each carry a `protocol`. It is an ordered list from the lowest layer to the highest, such as `[ethernet, ip]`. A single name, such as `protocol: modbus`, loads as a list of one entry.

```yaml
channels:
  - id: plc-01-to-control-lan
    channelType: network
    protocol: [ethernet, ip]
  - id: site-a-to-site-b-vpn
    channelType: virtual
    protocol: [ipsec, ip, tcp]
flows:
  - id: outstation-polling
    protocol: [dnp3]
```

The channel lists the layers that any traffic on it can use. The flow lists the protocol that only that flow uses, in the same way that an API which does one job is a flow. A hosting channel has no protocol.

Read the two lists together, with the channel first. The channel `[ethernet, ip]` and the flow `[dnp3]` give DNP3 over IP over Ethernet. Record the protocol in `protocol` so that a filter can find every channel that uses it, without a separate type for it.

### Vocabulary and layers

The stack picker in the channel, flow and network drawers offers a fixed vocabulary. Each entry belongs to one layer.

| Layer | Protocols | Position in the stack |
|---|---|---|
| Signalling | `ethernet`, `wlan`, `serial`, `lte`, `fibre-channel` | Optional. When present, it is the bottom entry. Each entry stands alone, so `wlan` needs no `ethernet` beneath it |
| Tunnel | `ipsec`, `gre`, `wireguard` | Above the entries it carries. A tunnel entry may follow an `ip` entry |
| Network | `ip` | Above signalling and tunnel entries |
| Transport | `tcp`, `udp` | Above the network entry |
| Application | `dnp3`, `modbus`, `iec-104`, `iec-61850`, `opc-ua`, `mqtt`, `https`, `ssh`, `snmp`, `ntp` | At the top |

Any other name is a valid entry. It has no layer, so the picker never warns about it. The picker writes each entry in lowercase and accepts the same entry twice.

### Order warnings

The picker warns when an entry sits below an entry of a higher layer, such as `ip` after `tcp`. The warning appears under the list, on the later entry of the pair.

A tunnel repeats the network layer, so `ip` followed by `ipsec` is not a warning. DNP3 over TCP over IP over IPsec over IP is the stack `[ip, ipsec, ip, tcp, dnp3]`.

The picker also warns about a second signalling entry and about any entry on a hosting channel. A warning is advice. The drawer saves the stack as written, and the model opens as before.

### Network default protocol

A network takes the same `protocol` list. It sets the default stack for the channels on that network, so the stack is written once.

```yaml
networks:
  - id: control-lan
    label: Control LAN
    protocol: [ethernet, ip]
```

A channel with no `protocol`, or an empty one, uses the default of the network at its end. A `protocol` on the channel always wins. A device connection to the network with no `protocol` also uses it.

A hosting channel never inherits. A channel between two networks with different defaults inherits neither. The model file keeps the channel as written. The channel drawer shows the inherited stack as a hint, and the protocol filters and the **IP Channels** option read it.

## Derived topology

Tetra derives the topology of a channel from the model, and the model does not store it. A channel is `multipoint` when one of its ends is a network node or when it joins more than two nodes. Any other channel is `point-to-point`. A hosting channel has no topology.

Only a network node counts as a shared medium. A zone, a group, a device and an information item do not. A `nodeB` list of two or more targets gives a channel more than two ends.

There is no `point-to-multipoint` topology. A radio hub with several remotes is point-to-multipoint only from the hub. That describes the topology around the hub, not one link.

The channel drawer shows the topology as a read-only line. The data slice field `topology` filters on it.

## Protocol stack by hop

A flow can cross several channels, so it carries a different stack on each. The route of a flow is its `nodeA`, each node in `path` and its `nodeB`, in order. A hop is one consecutive pair on that route.

A hop takes the stack of the channel that joins its two nodes, followed by the protocol of the flow. The channel stack is the one the channel carries, including any stack it inherits from its network. A hosting channel is never the path a flow takes.

An information item stands for the device that owns it. A path node that is not in the model is skipped, and its neighbours join.

A `via` on a path node names the channel for the hop that arrives at that node, provided that channel joins the hop's two nodes. A `via` that names any other channel only borrows its waypoints to draw the line, and the hop takes the channel that joins its nodes.

A hop has no stack when no channel joins its nodes, or when more than one does. A flow with a list of targets in `nodeB` has no hops. Two devices on one network need the network in `path` to get hop stacks.

The flow drawer lists each hop with its channel and its full stack, from the lowest layer to the highest. The data slice field `hopProtocols` holds the distinct protocols across all hops of a flow. For example, `item.hopProtocols |> hasAny("ipsec")` selects the flows that cross a tunnel on some hop.

## Protocol boundaries

A protocol boundary is a node where the channel stack changes between two consecutive hops of a flow. For example, DNP3 runs over IP on one hop and over a serial line on the next. A node at that point converts between the two stacks, such as a terminal server or a protocol gateway.

A device at the boundary is read as able to convert, because the model has no field that marks a converter. Any other node at the boundary, such as a network, cannot convert. Tetra then adds a warning to the Problems panel that names the flow, the node and the two stacks. The model still opens.

A hop with no stack takes no part in the check, because no stack is known on that side. The same holds for a hop whose channel states no protocol. The flow drawer marks each boundary between the hops it lists.

## Line style and colour

The canvas draws a channel from its type: network solid, virtual in its own colour, and hosting dotted. The legend shows one row for each type in use.

Two optional fields override the type for one channel. `lineStyle` is `solid`, `dashed` or `dotted`. `lineColor` is a colour in `#RRGGBB` form. A template can set either field, and a channel that sets it again wins.

The **IP Channels** filter option selects the network channels whose protocol list includes `ip` or is empty. A channel with no list of its own counts the list it inherits from its network.

## Flow types

| Value      | Use it for                                                                                                            |
| ---------- | --------------------------------------------------------------------------------------------------------------------- |
| `system`   | Information about the state of the system, such as platform health, remote interactive access or system software changes |
| `function` | The functional information, such as process values, alarms or status                                                   |
| `security` | Information that supports security, such as authentication or logging                                                  |

The type sets the colour of the line, so a reader can follow one kind of information across the diagram. If a flow fits two types, choose the type of the information that gives the flow its purpose.

## Direction, initiation and criticality

Both kinds share three settings.

| Setting       | Values                                   | Meaning                                                                                                            |
| ------------- | ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `direction`   | `a-to-b`, `b-to-a`, `bidirectional`      | Which way the traffic or information moves. The default is `bidirectional`                                         |
| `initiation`  | `interactive`, `programmatic`, `offline` | Who starts it: a person, software or a manual transfer. The default is `programmatic` |
| `criticality` | `critical`, `important`, `normal`        | How much the system depends on it. The default is `normal`. The exact definition of each level is not settled yet |

## Paths

A channel or flow with no path draws a direct line. A `path` lists intermediate nodes that the line passes through, such as a firewall or a gateway. Add a hop when the line must route through a device that the relationship actually crosses. See [Path segments and the segment nudge popover](/docs/sightline/usage/tetra/path-segments).

## Channels, flows and zones

The model stores no separate trust boundary or conduit. A relationship is cross-zone when the nearest zone of its first node differs from the nearest zone of its second node. A relationship between a zone and a zone nested inside it also counts as cross-zone.

The starter rule package reports each cross-zone relationship as a record for every zone that it enters. A bidirectional relationship enters both zones, so it gives two records. The rule reads direction and zones, and it ignores the channel type and the flow type. See [Rule packages](/docs/sightline/usage/tetra/rule-packages) and [Rules and records](/docs/sightline/usage/tetra/rules-and-findings).

To control which relationships appear in a rule, place the nodes in the right zones. To control which appear on a diagram, use a [data slice](/docs/sightline/usage/tetra/data-slice-filtering).
