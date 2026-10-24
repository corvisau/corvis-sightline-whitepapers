A **requirement attribute** is a named, valued property of a requirement, such
as `mil: MIL3`, `progression-area: asset-inventory` or `deployed` owned by the
SCADA team.

A **tag** is a freeform label with no key and no value, such as `EDR` or
`do later`.

> **Authoring:** attribute sets, requirement attachment and pair-level overrides
> are all authored in the app - see [Authoring attributes and attribute sets](/docs/sightline/usage/metron/attribute-authoring).
> This page covers the model; that one covers where you edit it.

---

## What an attribute holds

An attribute has a name. It can also carry these properties, and all of them are optional:

| Property | Meaning |
|---|---|
| Value | The value of the attribute (`MIL1`). Leave it empty for a declaration rather than a measurement. |
| Label | Display name when it should differ from the name (name `mil`, label `MIL`). |
| Type | `indicator`, `activity`, `standard`, `assurance` or `governance`. With no type, the attribute is an ordinary descriptive attribute. A typed attribute is assessable: see [Per-attribute assessment](/docs/sightline/usage/metron/attribute-assessment). |
| Owner | Who is accountable. Free text, so a placeholder (`{system-owner}`) works as well as a literal (`SCADA team`). |
| Description | Human explanation shown alongside the attribute. |

An attribute with only a name declares that the attribute applies. Its value comes from elsewhere, or is not relevant.

**Attribute names must not be integer-like.** The app refuses a name such as `2`,
because a number-like name would silently reorder a checklist. The app otherwise
keeps the order in which you add attributes.

## Reusing attributes: attribute sets

When the same attributes apply to many requirements, such as one control
deployed across fifteen zones, create them once as an **attribute set**. Sets
live in an attribute-set package. A requirement package uses them when you add
that package on its **Packages** tab. See
[Authoring attributes and attribute sets](/docs/sightline/usage/metron/attribute-authoring).

A requirement attaches sets by name and adds or overrides attributes on top. For
example, a set can give `deployed` the type `indicator` and the owner
`{system-owner}`. A requirement that attaches the set can then set the owner of
`deployed` to `SCADA team`. The requirement keeps the type from the set.

### The one rule

A name that collides with an inherited attribute overrides it; a name that
does not is an addition. There is no separate "overrides" field.

Merging is per-attribute and deep, so overriding the owner keeps the type of the set.

An attached set that the package does not hold contributes nothing and does not
stop the package from loading. The package editor raises a warning.

## Target-state requirements have the same two fields

A **target-state requirement** is one pair of a requirement and an entity on the
assessment worklist. It carries the same attached sets and the same attributes
as the requirement, one layer down. A single zone can attach an
extra set, or override one attribute, without touching the requirement every
other zone shares.

The **Assess** drawer edits the overrides of a pair in its **Target state
overrides** section. A pair holds two things for this:

- The sets that this pair attaches, for example `target-2027`.
- The attributes that this pair says itself, for example `deployed` with the
  value `partial` and the owner "SCADA team".

### Precedence, lowest to highest

1. The sets that the requirement attaches, in attachment order (later wins)
2. The attributes of the requirement itself
3. The sets that the pair attaches, in attachment order (later wins)
4. The attributes of the pair itself

Each layer folds onto the one beneath it by the same rule: collision overrides,
no collision adds, per-attribute and deep. So a pair that supplies only an owner
keeps the type that the set of the requirement established.

### A pair stores only what it owns

The resolved map of all four layers, which the worklist, the drawer and the filters display, is
never stored. A pair stores only the sets it attaches and the attributes
it declares itself.

The reference to an attribute set therefore stays live. Edit the set and
every pair that inherits from it moves, because none of them captured a copy.

## Filtering on attributes

Attributes are not offered in the Basic filter tab, because they are a record
and not a scalar field. In the Advanced tab, a program reaches an attribute by
its path:

```
item.attributes.mil.value == "MIL3"
```

On the worklist, `item.attributes` is the resolved map of all four
layers of the pair. A filter therefore reaches an attribute that the requirement inherits
from a set, and one that the target state overrides. It also reaches attributes
that the requirement declares itself.

⛔ **Do not pipe an attribute path into `hasAny` / `hasAll` / `hasNone` /
`hasExact`** (or the lambda forms `anyOf` / `allOf` / `noneOf`). Those require an
array and return `false` for a record, with no error and no diagnostic, so the
filter silently matches nothing.

`tags` is an array, so the array transforms are correct there:

```
item.tags |> hasAny("EDR")
```

A Compliance slice matches attributes through the **Attributes** rows of its
editor. Select **New** next to the **Compliance slice** picker. Select **Add
attribute**. Enter a key and a value. See
[Worklist filtering](/docs/sightline/usage/metron/worklist-filtering).

A slice matches the resolved attributes of a requirement. So `mil: MIL3` arriving
through an attached set satisfies it exactly as an attribute that the requirement
declares itself does.

## Column visibility

The Attribute sets grid renders in two places: an attribute-set package's own
editor, and the read-only **Attribute sets** tab of a requirement package. Each
has its own **Edit columns** toolbar button to show, hide and reorder columns:

- The attribute-set package's own editor never lists **source** in its
  picker. An attribute-set package holds only its own sets, so the column
  would repeat one value. The requirement package's Attribute sets tab does
  list it, since its grid mixes sets from more than one package.
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
