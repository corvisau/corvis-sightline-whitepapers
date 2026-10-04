Tetra entities can inherit from a shared **template** instead of repeating a
full body, and can pull shared prose in with Handlebars partials. Lamina uses the
same engine.

## Declaring template files

Templates live in their own files, listed in the manifest:

```yaml
# files.yaml
blockdiagram:
  meta: { id: site-a, name: Site A, version: 1, title: Site A }
  treeRoot: root
files:
  - devices.yaml
templates:
  - devices.template.yaml
```

`templates:` is a list of manifest-relative paths and defaults to `[]`. A
declared file that is missing or contains invalid YAML produces a warning in
the Problems panel and is skipped. The rest of the model still loads.

## Inheriting from a template

A template is an entity with an id, declared in a template file:

```yaml
# devices.template.yaml
devices:
  - id: tpl-plc
    icon: plc.svg
    vendor: ExampleControls
    description: A programmable logic controller.
```

Reference it with `template:` and override whatever differs:

```yaml
# devices.yaml
devices:
  - id: plc-01
    template: tpl-plc
    label: PLC 01          # authored fields win
  - id: plc-02
    template: tpl-plc
    label: PLC 02
    vendor: OtherVendor    # overrides the template's value
```

Both devices get `icon` and `description` from `tpl-plc`. `plc-02` keeps its
own `vendor`.

Overrides always win. Merging is deep: a nested object in the template is
merged key by key with the instance's.

### Chaining

A template may itself declare `template:`. The nearest value wins:

```yaml
devices:
  - id: tpl-device-base
    vendor: ExampleControls
    icon: generic.svg
  - id: tpl-plc
    template: tpl-device-base
    icon: plc.svg          # beats the base's generic.svg
```

The merge stops at a circular chain (`tpl-a → tpl-b → tpl-a`) and reports a
warning.

## Authoring templates in the UI

Templates can also be edited from the **Templates** panel (command
palette -> **Tetra: Edit Templates**). See
[Tetra Template Editor](/docs/sightline/usage/tetra/template-editor). The panel types each template by
its top-level key. The `tpl-plc` example above is a device template because it
sits under `devices:`, and it lists and edits like any other template. The
recommended form is still `tpl-<type>-<slug>` (`tpl-device-plc`). The panel
mints that form for templates created there, and uses it to type a template
whose key it cannot read.

## Which entity types support `template:`

All eight entity types support it: zone, group, device, network, information,
channel, flow and system.

A templated entity needs only `id` and `template:`; every other field becomes
an optional override. For channels and flows this includes the endpoints, so a
template may supply `nodeA` / `nodeB`.

## ⛔ A template must not own `members`

The Problems panel reports a template that declares `members:` as a model
error, naming the template id and the key:

```yaml
# WRONG - every instance would clone these children
zones:
  - id: tpl-zone
    members: [dev-shared]
```

Members are id-bearing child nodes. Cloning them into every instance
duplicates their ids, which corrupts by-id reference resolution across the
whole model. Author members on each instance that uses the template.

## Handlebars partials

Any string field on a template is available as a partial, addressed
`{{> <template-id>.<field>}}`:

```yaml
devices:
  - id: plc-03
    label: PLC 03
    description: '{{> tpl-plc.description}}'
```

`{{parent.<field>}}` resolves against the containing node, which is useful for
inline members:

```yaml
zones:
  - id: z-control
    label: Control Room
    members:
      - type: device
        id: hmi-01
        label: HMI 01
        description: 'Located in {{parent.label}}'   # -> "Located in Control Room"
```

## Saving keeps your authored form

Editing a templated entity in Tetra does not flatten it. Save `plc-01` after
changing its label and the file still reads:

```yaml
devices:
  - id: plc-01
    template: tpl-plc
    label: PLC 01 (edited)
```

Authored `{{> partial}}` references are likewise written back verbatim rather
than as the expanded prose.

Two related consequences of the same mechanism:

- Long strings are never re-wrapped into folded block scalars on save.
- Saving does not write out schema defaults you never typed (`direction:
  bidirectional`, `tags: []`, `version: 1`). The defaults still apply when the
  model loads.

## Relationship to Lamina

Lamina supports `template:` inheritance and the same Handlebars partial syntax.
Tetra also ships the Template Editor panel ([Tetra Template Editor](/docs/sightline/usage/tetra/template-editor))
and the Constants Editor panel ([Tetra Constants Editor](/docs/sightline/usage/tetra/constants-editor)).
Lamina has the same two panels.
