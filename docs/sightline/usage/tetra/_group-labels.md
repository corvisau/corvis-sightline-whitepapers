A group must carry a non-empty `label`.

```yaml
groups:
  - id: grp-site-a-prod
    label: Site A Production      # required
    members: [dev-virt-host]
```

The **New group** dialog in Tetra already requires a name.

## What is and isn't affected

| Entity | `label` on the full (non-templated) form |
|---|---|
| **group** | **Required, non-empty** |
| zone, device, network, information, system | Required |
| **channel, flow** | **Optional** — see below |

A templated group inherits its label, so the sparse form stays optional:

```yaml
groups:
  - id: grp-site-a-prod
    template: tpl-group-standard   # label comes from the template
```

### Why channels and flows are different

A relationship authored as nothing but its endpoints is a supported form.
The loader assigns it a deterministic `C-XXXXX` / `F-XXXXX` service tag:

```yaml
channels:
  - nodeA: dev-app-server        # no id, no label: both derived
    nodeB: net-core
    type: network_ip
```

Tetra also synthesises unlabelled channels for device-to-network adjacency.
Requiring a label on relationships would delete both behaviours, so
`channel.label` and `flow.label` stay optional. Use `showLabel` to control
whether a relationship's label is drawn at all.
