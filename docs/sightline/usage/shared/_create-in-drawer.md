Creating an entity in Tetra or Lamina opens the props drawer in a new-entity mode. No separate dialog opens. The fields that you fill in on create are the fields that you see on a later edit, in the same order, with the same widgets.

## What the form shows

The field set comes from the entity's schema. A field appears on the create surface when its schema marks it visible there.

A required field shows the required bar and blocks **Create** until you fill it. The reason appears under the field.

## Annotations

An annotation is the one type whose form differs, because an annotation has no label. Every other type shows ID and Label. An annotation shows ID and a multiline **Value**, which is the bubble's text and is required.

An annotation has no parent picker, because nothing contains it. It links to what it annotates, and the entity that you right-clicked becomes that link. The destination-file picker opens on the file that holds that entity. You can send the annotation to a different file.

For the style, colour and icon fields, see [Annotation styling](/docs/sightline/usage/shared/annotation-styling).

## Finishing and abandoning

**Create** is the only way to commit a new entity. Nothing is written until you click it.

Closing the drawer without creating leaves the model untouched. The drawer asks first, and offers **Discard** or **Cancel**. An untouched create request closes at once, because nothing is lost.

## Tetra

Tetra creates zones, groups, devices, networks, systems, information items, channels, flows and annotations.

### Where creating starts

Every entry point opens the same drawer.

| Entry point | How |
|---|---|
| Canvas | Right-click the pane, then choose **Add Zone**, **Add Group**, **Add Device**, **Add Network**, **Add System**, **Add Information**, **Add Channel** or **Add Flow**. Right-click a node for the subset that the node can hold. A zone, group, device or network also offers **Add Channel from here** and **Add Flow from here**. |
| Table | The toolbar's **New** menu, a row's context menu, or a right-click on empty space in the table (Tetra) |
| Device drawer | **+ New channel** and **+ New flow** in a device's Relationships section |
| Canvas annotations | Right-click a node or an edge, then choose **Add Annotation…** |

### Form fields

- **ID** is pre-seeded with a unique, correctly formatted id, and you can edit it. Zones get `Z-XXXXX`, groups `G-XXXXX` and information items `I-XXXXX`. Devices, networks and systems get `D-XXXXX`, `N-XXXXX` and `S-XXXXX`. Channels and flows get a service tag derived from their content, with a `C-` or `F-` prefix. The form accepts any id-safe value that matches the type's prefix. It also accepts a device, network or system id with a `dev-`, `net-` or `sys-` prefix, such as `dev-site-a-app`.
- **Label** is required for zones, groups, devices, networks, information items and systems. It is optional for channels and flows. An unlabelled edge is a supported form, and the model synthesises some channels without a label.
- **Advanced fields** are not on the create form. A new entity starts on the defaults, so the collapsible Advanced section that the Edit drawer shows does not appear. Set those fields, such as a group's **Style** and **Member Filter**, in the Edit drawer after you create the entity. The layout direction, margins, minimum sizes and alignment of a zone, group, device, network or information item are not in Advanced at all. Set them in the Container Properties panel.
- **Source file** is not a create field. You choose the destination with the file control described below.

### Where the entity is stored

Three controls surround the form. **Parent** sits above it. **Create as inline member** and **Destination file** sit together below it, because the checkbox decides whether the file control shows.

- **Parent** chooses the container that the entity lands under. It is the same control as the **Parent** field in the Edit drawer, and it sits in the same place, above **ID** and **Label**. The Edit drawer of a zone, group or device shows it, and choosing a different container there moves the entity. Type to filter the containers, or select **Browse…** to pick one from a tree. The field shows the container's name. It starts with the place where you began the create: right-clicking a zone pre-selects that zone, and starting from an empty part of the canvas or the toolbar **New** menu selects the model root, shown as **(root: model name)**. In a Tetra model with a hidden root group, **(root: model name)** is that group, so the entity lands at the top level of the canvas, and the group does not appear a second time in the list. Choose a different container to place the entity under it at creation. Zones, groups, devices, networks and information items show this control. Information items can be placed under a device or a zone. The other types can be placed under a zone or a group. Systems are stored on their own, and channels and flows are relationships, so they do not show it. **Add to New Group** does not show it either, because the group lands beside the members that you selected.
- **Create as inline member** nests the entity inside its parent's YAML. Without it, the entity gets a typed entry of its own. Containers (zones and groups) default to inline. Leaves (devices and networks) default to a standalone entry. Turning the option on hides the destination file, because an inline member has no file of its own.
- **Destination file** chooses the `*.tetra.yaml` file that receives the definition. It defaults to the file where the entity's would-be siblings already live. It falls back to the parent's own file or the manifest, shown as **Store with parent**. You can also create a new file or import an existing file without leaving the drawer.

An information item created under an owner becomes a per-owner inline badge. Its stored id carries the owner as a prefix, for example `dev-a.I-3KNT7`. The form shows only the bare tag.

### Finishing

**Create** writes the entity and leaves the drawer open on it, switched to edit mode.

The drawer asks for **Discard** or **Cancel** when you close it, open a different entity, start another create, or switch surface or data slice.

### See also

- [Tetra templates](/docs/sightline/usage/tetra/templates) covers templated entities and inherited fields.
- [Network Addressing](/docs/sightline/usage/tetra/network-addressing) covers how to edit a network's address ranges after you create it.

## Lamina

Lamina creates causes, events, outcomes, controls and annotations.

### Where creating starts

| Entry point | How |
|---|---|
| Canvas | Right-click the empty canvas, then choose **Add cause**, **Add event** or **Add outcome**. Choose **Add Risk…** for the guided wizard. |
| Findings inbox | **Create cause from finding** opens the new-cause surface, prefilled from the finding. |
| Canvas | Right-click a cause or outcome, then choose **Add control**. |
| Canvas | Right-click a cause, event, outcome or control, then choose **Add annotation**. |
| Table | The toolbar's **New** menu, a row's context menu, or a right-click on empty space in the table. The empty-space menu offers **Add Risk…**, **Add cause**, **Add event** and **Add outcome**. It also offers **Edit…**, which opens the Model Properties drawer. |

See [Findings](/docs/sightline/usage/lamina/findings) for the findings inbox.

### Required fields

A field is required when the canonical model schema says that it cannot be absent. This includes one conditional rule. A label, a control type and a category are required unless a template supplies them.

### Annotations

There is no type selector in the create drawer. Creating always makes a text annotation. To switch an existing annotation to Markdown, right-click it and choose **Set type**.

Adding an annotation does not put an empty bubble on the canvas. The drawer holds the draft. Closing the drawer without creating leaves the model untouched, and the same **Discard** prompt as any other unsaved draft appears.

The canvas bubble is display-only. To edit an annotation, right-click it and choose **Edit**, or select it. The drawer opens on the annotation's text, style, colour, icon and tags together.

### A created entity is an unsaved change

Creating an entity stages it. It does not write to disk until you save. In the Table, the new row shows the same highlight as an edited cell, across the whole row, until you save or discard it. Any other pending change marks its row the same way.

### Where the entity lands

The **Destination file** picker below the form chooses the YAML file that receives the new entity. A control has no picker. Lamina always writes a control inline under the cause or outcome that owns it.
