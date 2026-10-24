A template is a shared body that Tetra entities inherit instead of repeating the same fields. A device template can supply an icon, a vendor and a description, and each device that inherits from it shows those values. Lamina uses the same template engine.

## Author templates

Edit templates in the **Templates** panel. Select **Templates** in the diagram toolbar, or run **Tetra: Edit Templates** from the command palette. See [Tetra Template Editor](/docs/sightline/usage/tetra/template-editor) for the list, the drawer, and the controls that add, delete, reorder and register template files.

A template can inherit from another template. Choose the parent in the optional template selector when you create the template. The nearest value wins: a template overrides the fields it takes from its parent. Tetra stops the merge at a circular chain and reports a warning.

## Which entity types support templates

All eight entity types support templates: zone, group, device, network, information, channel, flow and system. For channels and flows, a template may supply the two endpoints.

## Inherited fields

A field that an entity does not set shows the template value. The field is read-only and has an **Override** button. Select **Override** to edit the field for this entity. The entity then keeps its own value, and the template is unchanged.

Overrides always win. A nested object in the template merges key by key with the entity's own.

## A template does not own members

The Problems panel reports a template that has members as a model error, and names the template. Members are child nodes with their own ids. If you copy them into every entity that uses the template, you duplicate those ids and break reference resolution across the model. Add members to each entity that uses the template.

## Saving keeps your own values

When you edit an entity that inherits from a template, Tetra does not copy the template values into the entity. Tetra saves only what you set. Change the label of a device and save, and the device still inherits its other fields from the template.

## Related panels

Tetra also has the [Tetra Constants Editor](/docs/sightline/usage/tetra/constants-editor). Lamina has the same Template Editor and Constants Editor panels.
