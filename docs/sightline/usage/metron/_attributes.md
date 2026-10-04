A **requirement attribute** is a named, valued property of a requirement, such
as `mil: MIL3`, `progression-area: asset-inventory` or `deployed` owned by the
SCADA team.

A **tag** is a freeform label with no key and no value, such as `EDR` or
`do later`.

> **Authoring:** attribute sets, requirement attachment and pair-level overrides
> all have UI now - see [Authoring attributes and attribute sets](/docs/sightline/usage/metron/attribute-authoring).
> This page covers the shape and the model; that one covers where you edit it.

---

## The shape

Attributes are a record keyed by attribute name:

```yaml
requirements:
  - id: ASSET-1a
    version: 1
    title: Maintain an asset inventory
    scopeRuleRef: zones-inscope
    assessment: { method: manual, ruleRef: null }
    type: control

    attributes:
      mil:
        value: MIL1
      progression-area:
        value: asset-inventory

    tags: [EDR, do later]
```

Each attribute may carry `value`, `label`, `type`, `owner` and `description`,
all optional. `{ documented: {} }` is a valid bare declaration that the
attribute applies, with its value supplied elsewhere or simply not relevant.

| Field | Meaning |
|---|---|
| `value` | The attribute's value (`MIL1`). Absent for a declaration rather than a measurement. |
| `label` | Display name when it should differ from the key (key `mil`, label `MIL`). |
| `type` | `indicator` \| `activity` \| `standard` \| `assurance` \| `governance`. Absent = an ordinary descriptive attribute. A typed attribute is assessable: see [Per-attribute assessment](/docs/sightline/usage/metron/attribute-assessment). |
| `owner` | Who is accountable. Free text, so a placeholder (`{system-owner}`) works as well as a literal (`SCADA team`). |
| `description` | Human explanation shown alongside the attribute. |

**Attribute names must not be integer-like.** `2:` is rejected: JavaScript
sorts integer-like object keys ahead of every other key regardless of where they
were authored, which would silently scramble a checklist. Authored order is
otherwise preserved, through both loading and saving.

## Reusing attributes: attribute sets

When the same attributes apply to many requirements, such as one control
deployed across fifteen zones, author them once as an **attribute set**. Sets
live in `attributes/*.attributes.yaml`, which is glob-scanned like `rules/` and
needs no manifest entry:

```yaml
attributeSets:
  - id: edr-ot
    name: EDR for OT
    attributes:
      deployed:
        type: indicator
        owner: "{system-owner}"
      connected:
        type: indicator
```

A requirement attaches sets by id and adds or overrides on top:

```yaml
    attributeSetRefs: [edr-ot]
    attributes:
      deployed:
        owner: SCADA team      # overrides the set's owner, keeps its type
      mil:
        value: MIL3            # not in any set - an addition
```

### The one rule

A name that collides with an inherited attribute overrides it; a name that
does not is an addition. There is no separate "overrides" field.

Merging is per-attribute and deep, so overriding `owner` keeps the set's `type`.

An `attributeSetRefs` id that matches no set contributes nothing and does not
fail the load. The package editor raises a warning.

## Target-state requirements have the same two fields

A **target-state requirement** is one `(requirement x entity)` pair on the
assessment worklist. It carries the same `attributeSetRefs` and `attributes`
fields as the requirement, one layer down. A single zone can attach an
extra set, or override one attribute, without touching the requirement every
other zone shares.

Pairs live in the assessment's `pairs:` list, in `*.assessment.yaml`. The
Assess drawer edits the overrides of a pair. To write them by hand, use YAML:

```yaml
pairs:
  - requirementId: ASSET-1f
    entityId: z-ot
    state: in-progress
    outcome: null
    evidence: []

    attributeSetRefs: [target-2027]     # what THIS pair inherits
    attributes:
      deployed:
        value: partial                  # what THIS pair says itself
        owner: SCADA team
```

### Precedence, lowest to highest

1. The requirement's attached sets, in `attributeSetRefs` order (later wins)
2. The requirement's own `attributes`
3. The pair's attached sets, in `attributeSetRefs` order (later wins)
4. The pair's own `attributes`

Each layer folds onto the one beneath it by the same rule: collision overrides,
no collision adds, per-attribute and deep. So a pair supplying only `owner`
keeps the `type` the requirement's set established.

### A pair persists only what it owns

The resolved four-layer map, which the worklist, drawer and filters display, is
never written back. What is saved to `*.assessment.yaml` is exactly the
two fields you authored above: the set ids the pair attaches, and the attributes
it declares itself.

The reference to an attribute set therefore stays live. Edit the set and
every pair inheriting from it moves, because none of them captured a copy.

## Filtering on attributes

Attributes are not offered in the Basic filter tab, because they are a record
and not a scalar field. Hand-authored Advanced-tab programs reach them by direct
path:

```
item.attributes.mil.value == "MIL3"
```

On the worklist, `item.attributes` is the pair's resolved map of all four
layers. A filter therefore reaches an attribute that the requirement inherits
from a set, and one that the target state overrides, as well as attributes typed
inline.

⛔ **Do not pipe an attribute path into `hasAny` / `hasAll` / `hasNone` /
`hasExact`** (or the lambda forms `anyOf` / `allOf` / `noneOf`). Those require an
array and return `false` for a record, with no error and no diagnostic, so the
filter silently matches nothing.

`tags` is an array, so the array transforms are correct there:

```
item.tags |> hasAny("EDR")
```

Compliance slices match attributes through their own `attributes:` predicate
list:

```yaml
slices:
  - id: mil3-only
    attributes: [{ key: mil, value: MIL3 }]
```

A slice matches a requirement's resolved attributes, so `mil: MIL3` arriving
through an attached set satisfies it exactly as an inline `mil: MIL3` does.

## Column visibility

The Attribute sets grid renders in two places: an attribute-set package's own
editor, and the read-only **Attribute sets** tab of a requirement package. Each
has its own **`[Edit columns]`** toolbar button to show, hide and reorder columns:

- The attribute-set package's own editor never lists **source** in its
  picker: an attribute-set package holds only its own sets, so the column
  would always read `this package`. The requirement package's Attribute sets
  tab does list it, since its grid mixes sets from more than one source.
- Drag the handle at the left of a column header to move that column. To use the keyboard, focus the handle, press Space, press the left or right arrow key, then press Space again. The picker shows the same order, and a drag never sorts the column. The handle works in both hosts.
- Your choice is personal and local. Hiding a column in either
  host is reflected the next time either mounts. Toggling a column from the attribute-set package's own editor never
  disturbs a **source** visibility choice made from the requirement package's
  Attribute sets tab.
- Dragging a column's resize handle saves its width the same way.

## Related

- [Per-attribute assessment](/docs/sightline/usage/metron/attribute-assessment) - recording what the
  assessor found for each typed attribute.
- [Worklist filtering](/docs/sightline/usage/metron/worklist-filtering) - Compliance slices and the
  Basic/Advanced filter tabs.
