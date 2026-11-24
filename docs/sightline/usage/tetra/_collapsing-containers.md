A zone or group on the Tetra canvas has a collapse control in its header. Each click moves the container to the next level. A zone has four stops, and a group has three.

| Click | Level | Applies to |
|---|---|---|
| 1 | Partial | Zones and groups |
| 2 | Full | Zones and groups |
| 3 | Flat | Zones only |
| Next | Expanded | Zones and groups |

A group has no header bar to flatten into, so its third click expands it.

## Collapse Levels

**Partial.** The container keeps its layout and shows each device as a small box. The container drops networks and device images.

**Full.** The container becomes a box that holds one stacked device list. The list names the devices inside and shows their information items as badges. A badge appears once, however many devices carry it.

**Flat.** A zone becomes a single coloured rectangle with its label. The rectangle holds no device list.

## Connection Endpoints on a Collapsed Container

Each level moves the end of a flow or channel that reaches inside the container.

| Level | A connection to something inside ends on |
|---|---|
| Partial | The small box of the device it belongs to |
| Full | The border of the container box |
| Flat | The border of the rectangle |

At the full level, the connection ends on the container and not on the device list. A connection that reaches an information item inside the container also ends on the container border. The device list carries no ports and does not grow with the number of connections. The container grows tall enough to give each connection its own port on the border.

A flow whose two ends are both inside one full or flat container is not drawn. A channel that a device creates through its `connections` field is not drawn when an end sits inside a collapsed container. A channel that an author declares directly stays and ends on the container border.

An annotation on a device inside a full or flat container moves to the container box and stays visible.

A group set to `hidden` draws no box. At the full level, its connections end on its device list instead.

## Saving and Exporting

A data slice saves the collapse level of each container with its view options. A PNG export draws each container at the level it has on the canvas.
