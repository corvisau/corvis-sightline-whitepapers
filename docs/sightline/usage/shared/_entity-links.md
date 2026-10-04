An entity can carry a list of links. Each link points to another entity in a model. A link can also point to an external record such as a Jira ticket, a GitHub issue or a web page. The props drawer edits the list in Tetra, Lamina and Metron.

The Markdown links in a text field are a separate feature. See [Markdown links](/docs/sightline/usage/shared/markdown-links).

## What a link holds

A link has five fields.

| Field | Meaning |
|---|---|
| Relationship | How this entity relates to the target. |
| Target kind | What the target is. |
| Target | The target's identifier: an entity id, an issue key or a URL. |
| Name | An optional display name. The Links column shows the target when the name is empty. |
| Instance | An optional server or model that holds the target, such as a Jira host. |

The relationship and the target kind accept any text. The lists in the drawer suggest common values.

| Field | Suggested values |
|---|---|
| Relationship | `relates-to`, `implements`, `depends-on`, `blocks`, `duplicates`, `mitigates`, `evidences` |
| Target kind | `entity`, `http`, `jira`, `github`, `file` |

## Edit the links of an entity

1. Open the props drawer for the entity.
2. Find the **Links** field.
3. Click **Add link**.
4. Choose or type a relationship and a target kind.
5. Type the target.
6. Save the change as you save any other edit.

A new row starts as an `entity` link with the relationship `relates-to`. To remove a link, click the **×** button at the end of its first line.

A row is saved only when the relationship, the target kind and the target all have a value. An incomplete row shows a note under it and is not written to the file. Closing the drawer discards an incomplete row.

When you remove the last link, the entity has no `links` entry in its file.

## Table columns

The Tabular Editor in Tetra and Lamina offers a **Links** column. Add it with **Edit columns**. Each cell lists the first three links by name, or by target when a link has no name. A count of the remaining links follows. The cell is read-only. Edit the links in the props drawer.

The Metron requirement, rule and attribute set tables show a **Links** column by default.

## Entities that carry links

| App | Entities |
|---|---|
| Tetra | Devices, zones, groups, networks and information items |
| Lamina | Causes, events, outcomes and controls, and the actors, vectors, exposures and inherent controls inside them |
| Metron | Requirements, rules and attribute sets |

A Lamina template can carry links, and an entity that uses the template shows them as inherited. Click **Override** to replace them.

## Tetra and Lamina

For a link with the target kind `entity`, the target box searches the objects in the open model by id or label. Choose one from the list, or type an id.

Under the row, a line tells you what the id matches:

- **Opens** and the type and id of the object, with an **Open** button that switches the drawer to that object
- **No object has this id**
- **Several objects have this id**

The check does not stop you from saving a link to an id that does not exist.

## Metron

The target box is plain text. Metron does not look up an entity target, so no line appears under the row.

## Limits

- An external link is stored and shown, but the drawer cannot open it.
- An entity link resolves only against the model open in the same app. A link from one app to an entity in another app is stored, but not followed.
- A Metron finding can carry links in its file. The finding drawer does not edit them.
