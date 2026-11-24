An entity can carry properties of your own. A property is a named value, such as an owner, a criticality number or a review flag. The props drawer edits the list in the main part of the form, above **Advanced**, and the model file keeps it in a `properties` record on the entity.

Tags are a separate feature. A tag is a short label with no value, and a member filter can match on it.

## Property Keys and Value Types

A property has a key and a value. The value has one of four types.

| Type | Label in the drawer | Example value |
|---|---|---|
| Text | **Text** | `Example Operations` |
| Whole number | **Integer** | `3` |
| Boolean | **Yes/No** | `true` |
| List of text items | **List** | `[insider, apt]` |

An **Integer** row refuses a decimal such as `1.5` and saves nothing.

A **List** row shows its items as chips. Type an item in the box on the row and press Enter to add it. Click the **×** on a chip to remove it. An item is text with no space at either end, and a list holds each item once. A list can be empty.

A key is unique on its entity, and upper and lower case are different. `Owner` and `owner` are two keys. A key cannot be empty, and the drawer removes a space at either end.

In the model file the key is the name and the YAML value is the type. This example is a Tetra device; a Lamina cause, event, outcome or control holds the same record:

```yaml
devices:
  - id: example-plc-1
    label: Example PLC 1
    properties:
      owner: Example Operations
      criticality: 3
      reviewed: true
      actor: [insider, apt]
```

Text that looks like a number or a boolean stays text while it is quoted. Choose **Text** for such a value, and the app writes `"3"` with the quotes.

## Edit the Properties of an Entity

1. Open the props drawer for the entity.
2. Find the **Properties** field.
3. Type a key in the box that reads **Add property and press Enter**, then press Enter.
4. Choose a type for the new row. A new property starts as **Text** with an empty value.
5. Type the value, tick the box for a **Yes/No** property, or add chips to a **List** property.
6. Save the change as you save any other edit.

The box suggests every key that another entity in the open model already uses. Choose a suggestion, or type a new key.

A row shows a note under it, and saves nothing, when the value does not fit its type. The note reads **Enter a whole number** or **Enter true or false**.

To change the type of a row, choose another type in its type list. A value that fits the new type converts. A text row with no value takes the default of the new type: `0` for **Integer**, `false` for **Yes/No** and an empty list for **List**. Text becomes a list by splitting on commas, and a list becomes text by joining its items with a comma and a space. A list does not become an integer or a yes/no value. Text that does not fit stays as it is, and the row shows the note.

To remove a property, click the **×** button at the end of its row.

## Templates

A template can set properties, and an entity that uses the template inherits them. The drawer lists an inherited key with the label **inherited**.

- To override an inherited value, change it in the drawer. The entity then holds its own value for that key.
- To go back to the template value, click **Reset** on the row of an overridden key.
- A list in a template is a default. An entity that sets its own list for that key replaces the whole template list. An entity with an empty list for the key has no items, so remove every chip to drop all the inherited items. Click **Reset** to inherit the template list again.
- An inherited key has no **×** button. Remove it in the template.

The Template Editor edits the same **Properties** field, with the same suggestions.

## Property Display in the Props Drawer

The props drawer is the only place that shows them. The Tabular Editor has no **Properties** column, and filters, bulk edit, reports and exports do not use properties.

The Create drawer does not offer the field. Add the properties after you create the entity.

## Tetra

All eight entity types carry properties: zones, groups, devices, networks, information items, channels, flows and systems. Annotations do not.

The suggestions come from all of these types, so a key used on a device is also offered on a flow.

## Lamina

Four entity types carry properties: causes, events, outcomes and controls. The inherent controls and conditional controls, and annotations do not.

```yaml
causes:
  - id: C001
    label: Operator opens the wrong valve
    properties:
      owner: Plant operations
      criticality: 3
```

### Properties on the Cause Card

The cause card in the bow tie shows three cells that read from a cause: **Actor**, **Exposure** and **Vector**.

| Cell | Property | What it shows |
|---|---|---|
| **Actor** | `actor` | The items of the property. |
| **Exposure** | `exposure` | The source zone of a cause that links Tetra flows. For any other cause, the items of the property. |
| **Vector** | `vector` | The names of the Tetra flows that the cause links. For any other cause, the items of the property. |

Each of the three properties takes a text value or a list of text items. A list shows its items joined with a comma and a space. A cell with no value shows a dash. The draw.io export writes the same cells.

A cause that links Tetra flows always shows the zone and the flow names, even when it also holds an `exposure` or `vector` property. The property never changes the source zone that the `exposure-grouped` likelihood mode groups by, so a cause with no flow link stays in the shared group.

```yaml
causes:
  - id: C006
    label: Harmful action by an authorised person
    properties:
      actor: [Insider]
      vector: Direct access
      exposure: Test
```

The suggestions come from these four types in the open model, so a key used on a cause is also offered on a control. They do not include Tetra keys.

A Lamina template can set properties for a cause, event, outcome or control. The drawer shows an inherited key with the label **inherited**, and **Reset** returns an overridden key to the template value, as the Templates section above describes.
