An entity can carry a list of links. Each link points to another entity in a model. A link can also point to an external record such as a Jira ticket, a GitHub issue or a web page. The props drawer edits the list in Tetra, Lamina and Metron.

The Markdown links in a text field are a separate feature. See [Markdown links](/docs/sightline/usage/shared/markdown-links).

## Entity Link Fields

A link has five fields.

| Field | Meaning |
|---|---|
| Relationship | How this entity relates to the target. |
| Target kind | What the target is. |
| Target | The target's identifier: an entity id, an issue key or a URL. |
| Name | An optional display name. The Links column shows the target when the name is empty. |
| Instance | An optional server or model that holds the target, such as a Jira host or a Tetra model. |

The relationship and the target kind accept any text. The lists in the drawer suggest common values.

| Field | Suggested values |
|---|---|
| Relationship | `relates-to`, `implements`, `depends-on`, `blocks`, `duplicates`, `mitigates`, `evidences`, `zone` |
| Target kind | `entity`, `http`, `jira`, `github`, `file` |

## Edit the Links of an Entity

1. Open the props drawer for the entity.
2. Find the **Links** field.
3. Click **Add link**.
4. Choose or type a relationship.
5. Type the target.
6. To set the target kind or a name, click **Details** on the link. In Tetra and Lamina, an `entity` link can show its instance box on the line itself, as described under **Tetra and Lamina**.
7. Save the change as you save any other edit.

Each link is one line: the relationship and the target. A link with a target kind other than `entity` shows the kind as a small label next to the target. **Details** opens a second line with the target kind, the name and, where the line has no instance box, the instance. Click **Details** again to close it.

A new link starts as an `entity` link with the relationship `relates-to`. To remove a link, click the **×** button at the end of its line.

To change the order of the links, drag a link by its handle at the start of the line. You can also use the keyboard: focus the handle, press Space to pick the link up, press the Up or Down arrow key to move it, and press Space to drop it. The order is saved with the entity.

A link is saved only when the relationship, the target kind and the target all have a value. An incomplete link shows a note under it and is not saved. Closing the drawer discards an incomplete link.

## Table Columns

The Tabular Editor in Tetra and Lamina offers a **Links** column. Add it with **Edit columns**. Each cell lists the first three links by name, or by target when a link has no name. A count of the remaining links follows. The cell is read-only. Edit the links in the props drawer.

The Metron requirement, rule and attribute set tables show a **Links** column by default.

## Entities That Carry Links

| App | Entities |
|---|---|
| Tetra | Devices, zones, groups, networks and information items |
| Lamina | Causes, events, outcomes and controls, and the inherent controls inside them |
| Metron | Requirements, rules and attribute sets |

A Lamina template can carry links, and an entity that uses the template shows them as inherited. Click **Override** to replace them.

## Tetra and Lamina

For a link with the target kind `entity`, the target box searches the objects in the open model by id or label. Choose one from the list, or type an id.

In Lamina, the line also has an instance box, before the target. Leave it empty for an object in the open model. Click it to choose a Tetra model of the workspace, listed with the hint `tetra`. The target list then shows only the zones of that Tetra model, and choosing a zone fills the target. With the relationship `zone` and an empty instance, the target list shows every zone of every Tetra model, and choosing one also fills the instance.

Under the row, a line tells you what the id matches:

- **Opens** and the type and id of the object, with an **Open** button that switches the drawer to that object
- **Zone** and the zone id and the Tetra model, such as `Zone z-3 in plant-b`, with no **Open** button
- **No object has this id**
- **No object in** the Tetra model **has this id**, when the instance names a Tetra model
- **Several objects have this id**

The check does not stop you from saving a link to an id that does not exist.

## Metron

The target box is plain text. Metron does not look up an entity target, so no line appears under the row.

## Limits

- An external link is saved and shown, but the drawer cannot open it.
- An entity link resolves only against the model open in the same app. A link from one app to an entity in another app is saved, but not followed.
- In Lamina, a link whose instance names a Tetra model is checked against that model's zones, and a Tetra zone is not followed. A link with no instance is checked only against the open Lamina model, so a zone link typed without an instance reads **No object has this id**.
- A link in a Markdown field names an id only. The picker that inserts a link into Markdown lists the objects of the open model, not Tetra zones.
- The Metron finding drawer does not edit links.
