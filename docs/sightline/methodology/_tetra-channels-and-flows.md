A channel and a flow both join two nodes, and they answer different questions. A channel is a conduit that could carry information flows between two nodes. A flow states that information moves between them. A model can hold a channel with no flow, a flow with no channel, or both between the same pair.

## Channel Against Flow

| | Channel | Flow |
|---|---|---|
| Represents | A conduit that could carry flows | Information that moves |
| Joins | Any two nodes | Information items, or devices when no information is named |
| The kind is set by | `channelType` | `type` |
| Line style shows | The channel type, or `lineStyle` | How the flow starts: `interactive` or `programmatic` |
| Line colour is set by | The channel type, or `lineColor` | The flow type |

A channel carries other flows and holds no information of its own. An API that other flows could use as a general carrier is a channel. An API that does one job is a flow.

A flow does not name a channel, and a channel does not name a flow. A path step on a flow can follow a channel, and that only changes how the line is drawn.

Model the channel first, then add flows for the information that matters. A channel with no flows still tells a rule that information could move between two nodes. A flow with no channel records information movement that the model does not tie to a particular conduit.

## Channel Types

A channel has a `channelType` that says what kind of conduit it is.

| Value | Shown as | Use it for |
|---|---|---|
| `network` | Network | A conduit over a network or a direct link. Most channels use this value, and it is the default |
| `virtual` | Virtual | A tunnel with endpoints of its own that rides other channels, such as an IPsec VPN between two site firewalls |
| `hosting` | Hosting | A node that runs on another, such as a virtual machine on its hypervisor or a VLAN on a switch. The host has full access to the guest's memory, storage and processing. The canvas draws it dotted |

The channel type does not describe the physical medium, because the medium changes along a route.

A connection that carries no information is not a channel, so the model has no type for a cable or a mechanical coupling.

A channel also has a `status` of `online` or `offline`.

## Protocol Stack

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

### Vocabulary and Layers

The stack picker in the channel, flow and network drawers offers a fixed vocabulary. Each entry belongs to one layer.

| Layer | Protocols | Position in the stack |
|---|---|---|
| Signalling | `ethernet`, `wlan`, `serial`, `lte`, `fibre-channel` | Optional. When present, it is the bottom entry. Each entry stands alone, so `wlan` needs no `ethernet` beneath it |
| Tunnel | `ipsec`, `gre`, `wireguard` | Above the entries it carries. A tunnel entry may follow an `ip` entry |
| Network | `ip` | Above signalling and tunnel entries |
| Transport | `tcp`, `udp` | Above the network entry |
| Application | `dnp3`, `modbus`, `iec-104`, `iec-61850`, `opc-ua`, `mqtt`, `https`, `ssh`, `snmp`, `ntp` | At the top |

Any other name is a valid entry. It has no layer, so the picker never warns about it. The picker writes each entry in lowercase and accepts the same entry twice.

### Order Warnings

The picker warns when an entry sits below an entry of a higher layer, such as `ip` after `tcp`. The warning appears under the list, on the later entry of the pair.

A tunnel repeats the network layer, so `ip` followed by `ipsec` is not a warning. DNP3 over TCP over IP over IPsec over IP is the stack `[ip, ipsec, ip, tcp, dnp3]`.

The picker also warns about a second signalling entry and about any entry on a hosting channel. A warning is advice. The drawer saves the stack as written, and the model opens as before.

### Network Default Protocol

A network takes the same `protocol` list. It sets the default stack for the channels on that network, so the stack is written once.

```yaml
networks:
  - id: control-lan
    label: Control LAN
    protocol: [ethernet, ip]
```

A channel with no `protocol`, or an empty one, uses the default of the network at its end. A `protocol` on the channel always wins. A device connection to the network with no `protocol` also uses it.

A hosting channel never inherits. A channel between two networks with different defaults inherits neither. The model file keeps the channel as written. The channel drawer shows the inherited stack as a hint, and the protocol filters read it.

## Derived Topology

Tetra derives the topology of a channel from the model, and the model does not store it. A channel is `multipoint` when one of its ends is a network node or when it joins more than two nodes. Any other channel is `point-to-point`. A hosting channel has no topology.

Only a network node counts as a shared medium. A zone, a group, a device and an information item do not. A `nodeB` list of two or more targets gives a channel more than two ends.

There is no `point-to-multipoint` topology. A radio hub with several remotes is point-to-multipoint only from the hub. That describes the topology around the hub, not one link.

The channel drawer shows the topology as a read-only line. The data slice field `topology` filters on it.

## Protocol Stack by Hop

A flow can cross several channels, so it carries a different stack on each. The route of a flow is its `nodeA`, each node in `path` and its `nodeB`, in order. A hop is one consecutive pair on that route.

A hop takes the stack of the channel that joins its two nodes, followed by the protocol of the flow. The channel stack is the one the channel carries, including any stack it inherits from its network. A hosting channel is never the path a flow takes.

An information item stands for the device that owns it. A path node that is not in the model is skipped, and its neighbours join.

A path step that follows a channel names it for the hop that arrives at that step's node, provided that channel joins the hop's two nodes. A channel that does not join them only borrows its waypoints to draw the line, and the hop takes the channel that joins its nodes.

A hop has no stack when no channel joins its nodes, or when more than one does. A flow with a list of targets in `nodeB` has no hops. Two devices on one network need the network in `path` to get hop stacks.

The flow drawer and the channel drawer list the targets in `nodeB` as rows. **Add target** adds a row, and picking a node fills it. Drag a row by its handle to change the order, or grab the handle with Space and press the arrow keys. The × button on a row removes it. A node can appear in one row only, and the first row is the End row of the Path section.

When Tetra writes `nodeB`, it writes a list, even for one target. A file that holds a single id still loads.

The flow drawer lists each hop with its channel and its full stack, from the lowest layer to the highest. The data slice field `hopProtocols` holds the distinct protocols across all hops of a flow. For example, `item.hopProtocols |> hasAny("ipsec")` selects the flows that cross a tunnel on some hop.

## Protocol Boundaries

A protocol boundary is a node where the channel stack changes between two consecutive hops of a flow. For example, DNP3 runs over IP on one hop and over a serial line on the next. A node at that point converts between the two stacks, such as a terminal server or a protocol gateway.

A device at the boundary is read as able to convert, because the model has no field that marks a converter. Any other node at the boundary, such as a network, cannot convert. Tetra then adds a warning to the Problems panel that names the flow, the node and the two stacks. The model still opens.

A hop with no stack takes no part in the check, because no stack is known on that side. The same holds for a hop whose channel states no protocol. The flow drawer marks each boundary between the hops it lists.

## Line Style and Colour

The canvas draws a channel from its type: network solid, virtual in its own colour, and hosting dotted. The legend shows one row for each type in use.

A hosting channel ends in two markers that show which node hosts the other. A filled diamond marks the host, which is the first node of the channel. A hollow ring marks the guest, which is the second node. The markers take the colour of the line, including a `lineColor` override. Network and virtual channels end in plain dots. The draw.io export keeps the same two markers.

Two optional fields override the type for one channel. `lineStyle` is `solid`, `dashed` or `dotted`. `lineColor` is a colour in `#RRGGBB` form. A template can set either field, and a channel that sets it again wins.

## Routing

The optional `routing` field of a channel is `direct` or `avoid`. A channel without it is `avoid`: the canvas routes it around the zones, groups and networks in its way.

A `direct` channel is drawn as network plumbing. It runs straight from a device to its network and does not detour around the network or the zones that hold it. The canvas hides a `direct` channel while either end sits in a collapsed container, and a collapsed container does not count it when it sizes its connection points.

Use `direct` for the short link between a device and the network it sits on. Leave `avoid` for a channel that crosses the diagram.

## Linking a Device from the Device Drawer

The device drawer links a device to another node by creating a channel. There are two ways.

- **Networks.** Pick a network in the Networks field. The drawer adds a channel between the device and the network, in the device's own file.
- **Connect to.** In the Channels list, type the id of another node and choose **Connect**. The drawer adds a channel between the device and that node, in the device's own file.

A channel to a network is a `direct` channel and carries the network's default protocol stack, or `ip` when the network has none. A channel to any other node routes around obstacles. The drawer shows an error under the input when the id is empty, unknown, or the device's own id.

To remove a link, remove the network chip, or choose **×** on a channel row where the device is an end. The drawer then asks whether to **Keep Orphan** or **Delete Channel Entirely**. Keep Orphan leaves the channel in its file with the device's end empty, as described below. Delete Channel Entirely removes the channel. Cancel changes nothing.

## A Channel with an Unattached End

A channel can lose one of its ends and stay in its file. This happens when you remove a connection from a device's connection list and choose to keep the channel. The channel keeps the end that still names a node. The end that named the device is left empty.

The canvas does not draw a channel with an empty end. The Problems panel lists it as a warning that names the channel and the empty end. The channel still appears in the Channels list of the node at its other end, with the empty side marked `unattached`.

To reattach it, open the channel and choose a node for the empty end in the channel form. To remove it, delete the channel. A flow always needs both of its ends.

## Flow Types

| Value      | Use it for                                                                                                            |
| ---------- | --------------------------------------------------------------------------------------------------------------------- |
| `system`   | Information about the state of the system, such as platform health, remote interactive access or system software changes |
| `function` | The functional information, such as process values, alarms or status                                                   |
| `security` | Information that supports security, such as authentication or logging                                                  |

The type sets the colour of the line, so a reader can follow one kind of information across the diagram. If a flow fits two types, choose the type of the information that gives the flow its purpose.

## Direction, Initiation, Initiator and Criticality

A channel and a flow share one setting.

| Setting       | Values                            | Meaning                                                                                                            |
| ------------- | --------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `criticality` | `critical`, `important`, `normal` | How much the system depends on it. The default is `normal`. The exact definition of each level is not settled yet |

A channel has a `direction`: `a-to-b`, `b-to-a` or `bidirectional`. It says which way the traffic moves. The default is `bidirectional`.

A flow says which way the information moves with `infoDirection`. It takes the same three values, and the default is `bidirectional`. The canvas draws an arrowhead at each end that the information reaches.

A flow has two more settings. A channel starts no exchange of its own, so it has neither.

| Setting      | Values                       | Meaning                                                                                                                                              |
| ------------ | ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `initiation` | `interactive`, `programmatic` | Who starts the exchange: a person or software. The default is `programmatic`. It sets the line style. Model a manual transfer as `interactive`       |
| `initiator`  | `a`, `b`, `both`             | Which end opens the exchange. A flow can move information from A to B while B opens the connection. The setting is optional, and unset means not stated |

`initiator` and `infoDirection` are independent. A historian that polls a controller opens the exchange from the historian end, and the information moves from the controller to the historian.

The canvas draws an open circle on the line near each initiating end. A flow that states no `initiator` has no circle. The arrowheads show `infoDirection` only. The legend lists the circle as the Initiating end when a flow in the model states an initiator.

A channel file that still holds an `initiation` opens without it, and the key goes at the next save. For a flow file that still holds `direction`, or an `initiation` of `offline`, see [Compatibility](/docs/sightline/upgrade/compatibility).

## Paths

A channel or flow with no path draws a direct line. A `path` lists intermediate nodes that the line passes through, such as a firewall or a gateway. Add a hop when the line must route through a device that the relationship actually crosses. 

Each hop is either a node or a channel. A channel hop follows that channel to its other end. Several relationships can follow the same channel and split where it ends. For example, two hosting channels from one server can each name a shared network channel as their first hop. Both lines run along it to its far node, then each continues to its own end.

A channel with several far ends offers a hop toward each end. The hop picker lists the channel once for every end it reaches from the previous hop. A hop toward an end that the channel does not draw a line to follows a direct run to that end.

See [Path segments and the segment nudge popover](/docs/sightline/usage/tetra/path-segments).

## Channels, Flows and Zones

The model stores no separate trust boundary or conduit. A relationship is cross-zone when the nearest zone of its first node differs from the nearest zone of its second node. A relationship between a zone and a zone nested inside it also counts as cross-zone.

The starter rule package reports each cross-zone relationship as a record for every zone that it enters. A bidirectional relationship enters both zones, so it gives two records. The rule reads direction and zones, and it ignores the channel type and the flow type. See [Rule packages](/docs/sightline/usage/tetra/rule-packages) and [Rules and records](/docs/sightline/usage/tetra/rules-and-findings).

To control which relationships appear in a rule, place the nodes in the right zones. To control which appear on a diagram, use a [data slice](/docs/sightline/usage/tetra/data-slice-filtering).
