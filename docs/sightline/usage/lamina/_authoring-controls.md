A cause or an outcome owns its controls. You write each control inline, in the
parent's `controls:` list. Reuse comes from a template.

This page covers the control fields a model author writes by hand. For the
effectiveness ratings and the zone rules that colour a control's element badges,
see [Lamina Data Slices](/docs/sightline/usage/lamina/data-slices).

## A control is always inline

Write the control inside the cause or the outcome that owns it:

```yaml
causes:
  - id: STN-NET-LTE-A
    label: LTE modem compromise
    controls:
      - id: K-MMM44
        label: Network segregation
        category: Technology
        controlType: Preventative
        currentEffRating: Full
```

Every control needs an `id`. The id must be unique across the whole model,
because two parents never share one control. Use the `K-#####` form for a new
control.

An outcome takes the same shape. Its controls carry `controlType: Mitigative`
instead.

## Reuse comes from a template

Put the shared part of a control in a template, then point each instance at it:

```yaml
# controls.template.yaml
controls:
  - id: tpl-control-generic-edr
    label: Endpoint detection and response
    category: Technology
    controlType: Preventative
```

```yaml
# causes.yaml
    controls:
      - id: K-PMKH3
        template: tpl-control-generic-edr
        currentEffRating: Partial
```

The loader merges the template under the instance. The instance wins on any
field it sets, so `currentEffRating` above overrides the template's value and
`label` comes from the template.

Keep the instance sparse. Write only the fields that differ from the template.
An instance that repeats the template's own values is harder to change later,
because a template edit no longer reaches it.

## What still uses an id reference

Two fields hold a reference rather than an object:

- `cause.events` names the events a cause can lead to
- `event.outcomes` names the outcomes an event can produce

Several causes point at one event, and several events point at one outcome.
Those are shared nodes, so they stay references.

## Elements — the IT/OT elements a control covers

A control's `elements` field names the assets it protects. Each entry is one
of three shapes:

```yaml
controls:
  - id: K-MMM44
    label: Network segregation
    elements:
      - IAM-1                      # wildcard: covers this element in every zone
      - [IAM-2, Z-SCADA]            # narrowed to one zone
      - [IAM-3, [Z-SCADA, Z-FIELD]] # narrowed to several zones
```

An entry can also be a full object (`{ id: IAM-4, name: ..., description: ... }`)
when you need to record detail beyond the id. The drawer preserves that detail
on round-trip, but the editor only exposes the id and the zone narrow.

The props drawer's Elements field lists one row per entry: an id and its zone
narrow (blank means every zone). **+ Add element** appends a row and **×**
removes one.

### Zone ids name tetra zones

A zone id on a cause, outcome, exposure, objective, control, element narrow or
vector `sourceZones` is the id of a zone in your tetra model. Lamina accepts any
non-empty string. Lamina has no `Zone` entity of its own.

Lamina checks zone ids when it opens a `*.bowtie.yaml` manifest or saves a yaml
file beside it. It adds a Problems-panel warning (`unresolved-zone-ref`) to the
manifest for each zone id that no tetra model defines.

The check reads the tetra models (`*.tetra.yaml` and the files each lists) under
the manifest's directory. A tetra model outside that directory is not seen, so
its zone ids report as unresolved.

## A template can derive from another template

Reuse is hierarchical. Write the shared part once, then derive the specific
case from it:

```yaml
# controls.template.yaml
controls:
  - id: tpl-control-generic-edr
    label: Generic EDR control
    category: Technology
    controlType: Preventative

  - id: tpl-control-edr-zone-x
    template: tpl-control-generic-edr
    label: EDR product A used in Zone X
    currentEffRating: Full
```

An instance points at the specific template and inherits through the whole
chain:

```yaml
    controls:
      - id: K-PMKH3
        template: tpl-control-edr-zone-x
```

That control takes `category` and `controlType` from the generic template,
`label` and `currentEffRating` from the specific one, and each level overrides
the one above it. A chain can be any depth.

Author the hierarchy in YAML. The Template Editor edits a template's own
fields; it does not create or re-parent a sub-template.

## Where an inherited value came from

The props drawer marks an inherited field with an `inherited` badge. Hover the
row to see which templates the value came through:

```
Generic EDR control -> EDR product A used in Zone X -> this control
```

The chain reads left to right, from the template that introduced the value to
the control in front of you. A field the nearest template overrides names only
that template.

The picker's **Browse** button shows the same hierarchy as a tree, with each
sub-template under its parent. Typing in the field filters the tree and keeps a
match's parents visible.

A template that derives from a template of a different type appears in the
picker as a top-level entry with no warning. An example is a control template
that names a cause template. Check the parent's type when a sub-template does
not appear where you expect it.
