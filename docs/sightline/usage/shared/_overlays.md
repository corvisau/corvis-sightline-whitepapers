An overlay is a render-only shaded region drawn on top of the diagram around a set of referenced entities. Overlays are non-interactive: they cannot be clicked or dragged. Authors write them in YAML. Neither Tetra nor Lamina has a create or edit UI for overlays.

The schema, modes, file placement and styling rules are the same in Tetra and Lamina. Only the entity kinds that an overlay can enclose, the stacking position and the examples differ. See the [Tetra](#tetra) and [Lamina](#lamina) sections.

---

## Schema

```yaml
overlays:
  - id: <string>           # required, unique within the model
    members: [<id>, ...]   # required, entity IDs to enclose (see below)
    label: <string>        # optional: shown in top-left corner in the fill colour
    mode: bounds | hull    # optional, default: bounds
    fill: <hex>            # optional, default: #1971c2 (blue)
    opacity: <0–1>         # optional, default: 0.18
```

### Fields

| Field | Required | Default | Notes |
|---|---|---|---|
| `id` | Yes | none | Must be unique within the file and model |
| `members` | Yes | none | IDs of entities to enclose; unknown IDs are silently skipped |
| `label` | No | *(none)* | Short text; rendered top-left in the fill colour |
| `mode` | No | `bounds` | `bounds` = bounding rectangle; `hull` = tight rectilinear polygon |
| `fill` | No | `#1971c2` | Any CSS hex colour |
| `opacity` | No | `0.18` | Fill opacity; stroke is `opacity × 2.5` (capped at 1.0) |

---

## Members

The `members` list takes entity IDs. The entity kinds that an overlay can enclose depend on the app. See the [Tetra](#tetra) and [Lamina](#lamina) sections.

The overlay ignores a member that the current diagram does not contain. An author can therefore write an overlay before all of its members exist. Several overlays can reference the same entity.

---

## Modes

### `bounds` (default)

The `bounds` mode draws a single rectangle that fits tightly around all members, with 12 px of padding.

```
┌─────────────────────────────┐
│  ┌──────┐       ┌──────┐    │
│  │ dev1 │       │ dev2 │    │
│  └──────┘       └──────┘    │
└─────────────────────────────┘
```

### `hull`

The `hull` mode draws a rectilinear (orthogonal) polygon that follows the outline of each member individually and merges overlapping shapes. Use `hull` when members are spread across the canvas and a single bounding box would enclose too much empty space.

```
┌────────┐
│  dev1  │
└────────┘──────────┐
              │ dev2 │
              └──────┘
```

---

## File placement

Put the `overlays:` block in any YAML file that the `files:` array of the manifest lists or matches with a glob. A dedicated `overlays.yaml` is the convention.

The examples for each app are in the [Tetra](#tetra) and [Lamina](#lamina) sections.

---

## Tips

- Omit `fill` to get the default blue (`#1971c2`). Omit `opacity` to get `0.18`, which is translucent but clearly visible.
- If an overlay is invisible, check that at least one `members` ID exists in the current diagram. The renderer skips an overlay with no resolved members.
- Use `hull` for members that are spread across the canvas. It avoids the large empty rectangle that `bounds` draws around them.
- Use `bounds` for members that sit close together. A `hull` over a compact cluster adds a more complex outline and no benefit.
- Labels are optional. Leave them out for purely visual grouping.
- The default opacity of `0.18` keeps node labels readable through the overlay.

---

## Tetra

### Members

A Tetra overlay encloses any combination of `device`, `zone`, `group` and `network` IDs.

### File placement

```yaml
# overlays.yaml
overlays:
  - id: ov-dmz
    label: DMZ
    members: [dev-fw, dev-proxy, zone-dmz]
```

Reference the file from the manifest with a glob:

```yaml
# blockdiagram.yaml
files:
  - "**/*.yaml"
```

Or list the files explicitly:

```yaml
files:
  - m.tree.yaml
  - m.devices.yaml
  - overlays.yaml
```

### Stacking order

Tetra draws overlays at layer 11. That layer is above the zone, group, network and device fills (layers 1 to 5) and above the system overlays (layer 10). It is well below edges and annotations (layer 1000 and above). See [Tetra canvas stacking order](/docs/sightline/usage/tetra/canvas-z-order) for the full order.

### Examples

Simple bounding box. This example shades the three devices in the control network.

```yaml
overlays:
  - id: ov-control-net
    label: Control Network
    members:
      - dev-plc-01
      - dev-hmi
      - dev-historian
    fill: '#2f9e44'
    opacity: 0.15
```

Hull. This example highlights scattered devices in the same security zone.

```yaml
overlays:
  - id: ov-internet-facing
    label: Internet-facing
    members:
      - dev-dmz-web
      - dev-dmz-mail
      - dev-vpn
    mode: hull
    fill: '#c92a2a'
    opacity: 0.12
```

Multiple overlays in one file.

```yaml
overlays:
  - id: ov-ot-l3
    label: OT Level 3
    members: [dev-mes, dev-erp-gw]
    fill: '#e67700'
    opacity: 0.18

  - id: ov-ot-l2
    label: OT Level 2
    members: [dev-scada, dev-historian, dev-eng-ws]
    fill: '#1971c2'
    opacity: 0.18

  - id: ov-ot-l1
    label: OT Level 1 / Field
    members: [dev-plc-a, dev-plc-b, dev-rtu]
    fill: '#5f3dc4'
    opacity: 0.18
```

Enclose an entire zone.

```yaml
overlays:
  - id: ov-cloud
    label: Cloud Services
    members:
      - zone-aws-prod    # the zone itself
      - dev-alb
      - dev-rds
    mode: bounds
    fill: '#0c8599'
    opacity: 0.12
```

---

## Lamina

### Members

A Lamina overlay encloses any combination of `cause`, `event`, `outcome` and `control` IDs.

### File placement

```yaml
# overlays.yaml
overlays:
  - id: ov-initial-access
    label: Initial Access
    members: [cause-spearphish, cause-usb-drop]
```

Reference the file from the manifest with a glob:

```yaml
# bowtiemanifest.yaml
files:
  - "**/*.yaml"
```

Or list the files explicitly:

```yaml
files:
  - m.causes.yaml
  - m.controls.yaml
  - overlays.yaml
```

### Stacking order

Lamina draws overlays at layer 11. That layer is above the cause, event, outcome and control node boxes (default layer 0) and below annotations (layer 50).

### Examples

Highlight a threat path.

```yaml
overlays:
  - id: ov-initial-access
    label: Initial Access
    members:
      - cause-spearphish
      - cause-usb-drop
    fill: '#c92a2a'
    opacity: 0.15
```

Group related controls.

```yaml
overlays:
  - id: ov-detection-controls
    label: Detection
    members:
      - ctrl-siem-alert
      - ctrl-edr
      - ctrl-ndr
    fill: '#2f9e44'
    opacity: 0.18
    mode: hull
```

Span multiple event types.

```yaml
overlays:
  - id: ov-lateral-movement
    label: Lateral Movement
    members:
      - event-pass-the-hash
      - event-wmi-exec
      - outcome-domain-admin
    fill: '#e67700'
    opacity: 0.14
```
