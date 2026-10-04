`zoneTrust` is a zone's display-only trust-level or classification label. It
renders as a small grey pill on the right of the zone's header bar, next to the
red `SL-T` pill. The draw.io export carries the same pill.

It has no structural role. Zone identity is `id`; nothing resolves,
references, or keys off `zoneTrust`. Two zones may carry the same value.

## Authoring

```yaml
zones:
  - id: sys-datacenter
    label: Data Center
    zoneTrust: Z-DC01
    slT: 3
    members:
      - grp-storage
```

Both pills are optional and independent. A zone with only `slT` renders one
pill, a zone with only `zoneTrust` renders one pill, a zone with neither
renders none. The header bar height is the same either way.

The field is equally available on a zone authored inline in a nested `tree:`
fragment:

```yaml
tree:
  - type: group
    id: root
    members:
      - type: zone
        id: sys-field
        label: Field Site
        zoneTrust: Z-FS01
```
