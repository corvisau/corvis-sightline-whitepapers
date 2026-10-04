This page covers the three places where a user authors attributes, in the order
a user meets them.

For what an attribute is and how the record shape works, see the
[Attributes, attribute sets and tags](/docs/sightline/usage/metron/attributes) page. For recording
current state against attributes, see the
[Per-attribute assessment](/docs/sightline/usage/metron/attribute-assessment) page.

---

## Two rules that surprise people

Both rules follow from the attribute record, which is keyed by name and folded
one layer onto the next:

1. **A colliding name overrides; it does not duplicate.** If a requirement
   declares `documented` and a set it attaches also declares `documented`, you
   get one attribute, not two. This is the override mechanism.
   Merging is per-property, so a layer supplying only `owner` keeps the earlier
   layer's `type`.
2. **An attribute with no `type` is never assessable.** The Assess drawer lists
   only attributes carrying a `type`. An attribute without one is descriptive
   (a `mil` level, a `progression-area`) and has no current state to find. If
   you author an attribute and it does not appear in the drawer, this is why.

The order attributes fold in:

```
1. the Requirement's attached AttributeSets, in attachment order
2. the Requirement's own attributes
3. the pair's attached AttributeSets, in attachment order
4. the pair's own attributes
```

The later layer wins a name collision, so attachment order is precedence order.

---

## 1. Author a reusable set in an attribute-set package

You author attribute sets in an **attribute-set package**, on its **Attribute
sets** tab. A requirement package uses them by naming that package in its
`uses:` list. See the [Component packages](/docs/sightline/usage/metron/component-packages) page.

| Column | Meaning |
|---|---|
| id | What `attributeSetRefs` points at. Immutable once created. |
| name | Human label, shown wherever the set is offered. Optional. |
| attributes | How many attributes the set carries. |
| used by | How many requirements attach it, across every requirement package that uses this one. |

- **+ Add attribute set** opens the **props drawer** as a create surface. Fill
  the id and an optional name. Add attribute rows. Then select **Create**.
- To edit a set, right-click its row. Select **Open**. The same menu carries
  **Delete**. A left-click on a row does nothing.
- The id is read-only once created. Every attachment points at it, and a
  rename in place would silently orphan them. To change an id, delete the set.
  Then add the set again.
- Each attribute row carries the full set of properties: name, value, label,
  type, owner, description.

A new set joins the package's first fragment file, or
`attributes/<packageId>.attributes.yaml` when the package has none.

### Deleting a set

Nothing blocks a delete. A requirement or a pair that attaches the set keeps the
reference, which then contributes no attributes. The loader raises a warning
rather than failing to load. Read the package's **References** tab before you
delete: it shows every requirement package that would be affected.

### Attribute sets in a requirement package

A requirement package's **Attribute sets** tab lists every set the package can
attach. The tab is read-only. It has no **+ Add attribute set** button or
drawer, and no delete.

- The **source** column names the attribute-set package each set comes from. A
  set from the package's own `attributes/` folder reads `this package`.
- To change a set from an attribute-set package, right-click it. Select
  **Open package**. The attribute-set package opens in its own editor.
- A set from the package's own folder has no action. To edit it, move it into
  an attribute-set package. The [Component packages](/docs/sightline/usage/metron/component-packages)
  page describes attribute-set packages.
- The **used by** column counts requirements in this package only.

A local set with the same id as a referenced one still overrides it for this
package alone.

## 2. Attach a set to a requirement

Open a requirement's props drawer. The **Attribute sets** field offers every set
in the package. An attached set folds its attributes in beneath the
requirement's own.

Attachment order is precedence: a set attached later wins a name collision with
one attached earlier. The field label says so, and the row list is not ordered
by precedence.

If you detach every set, the app removes the field rather than leaving an empty
list.

## 3. Override attributes on one pair — the Assess drawer

When you assess a pair, the drawer shows a **Target state overrides** section
above the per-attribute assessment.

- **Attribute sets for this pair** attaches sets at the pair level (layer 3).
- The rows beneath show what the pair inherits, and let you override any
  property of any of them (layer 4).

This section has two limits:

- **You cannot add a new attribute name here.** The rows are exactly what the
  pair inherits. There is no add-row control, and the name is read-only.
- **Only what you change is saved.** When you edit `owner` on one attribute, the
  pair stores that one property, not a copy of the whole attribute. Storing the
  inherited values verbatim would freeze a snapshot of the requirement, and a
  later edit upstream would stop reaching this pair.

If you edit a value back to what it inherits, the app removes the override
rather than pinning the current value.

The whole section is read-only once the pair is `done`. Reopen the pair to edit
the section.
