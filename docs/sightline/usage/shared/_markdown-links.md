A Markdown field in the props drawer can link to another object in the same model. Clicking the link opens that object in the drawer. Tetra and Lamina both support the feature, each over its own objects. Metron does not.

An annotation with `type: markdown` renders its `value` as Markdown on the canvas, and a link in it works the same way. See the [Annotations](/docs/sightline/usage/shared/annotations) page for the annotation types.

## Syntax

Write the object's id after `sightline:` as the link target.

- The id is the object's own `id` value, matched exactly and case-sensitively.
- The link carries the id only. It has no type prefix, so `sightline:O002` finds whichever object owns `O002`.
- Write a space in an id as `%20`.
- A target with no id, a `/`, a `?`, a `#`, or a malformed escape is not an object link. It renders as a link with no target.

Each app section below shows an example and lists the objects it can link to.

## Insert a link from the picker

The picker writes the link, so you do not type an id.

1. Click the field text outside any link. The editor opens.
2. Place the caret where the link goes. To reuse words as the link text, select them.
3. Click the `Insert link to object` button under the text. A search box and a list of objects appear.
4. Type part of an id or label to narrow the list.
5. Move with the arrow keys and press Enter, or click an option.

The picker inserts a link at the caret. With text selected, it replaces the selection and uses the selected words as the link text. A blank or multi-line selection is replaced with the object's label instead.

The list sorts each type by id. It renders the first 50 matches, and typing narrows the rest. Escape closes the list and returns focus to the text. It does not close the drawer.

The list leaves out an object whose id another object also owns, because a link to that id renders as broken. It also leaves out an id that a link cannot carry, such as one containing `/`.

## What the link does

| Link state | How it renders | On click or Enter |
|---|---|---|
| One object owns the id | Link colour, with the object's type, id and label in its tooltip | The drawer switches to that object |
| No object owns the id | Red wavy underline, with a tooltip that names the id | Nothing happens |
| Two or more objects own the id | Red wavy underline, with a tooltip that lists their types | Nothing happens |

Clicking a resolved link does not open the field's editor. To edit the text, click outside the link, or focus the field and press Enter or F2.

A resolved link opens the object through the same path as selecting it on the canvas. If a half-filled create draft is open, the unsaved-draft prompt appears first.

A broken link stays focusable, so a keyboard or screen-reader user can read its tooltip.

## Which fields

Every multiline field in the drawer renders as Markdown and supports links.

A link resolves against the model as staged in the drawer, whichever data slice is active. A link to an object that the slice hides still opens it.

An object you created and have not saved yet is linkable. An object you have staged for deletion stops resolving, and its links render as broken.

## Annotation bubbles on the canvas

A `sightline:` link in a `markdown` annotation uses the same syntax and shows the same three states as a link in a drawer field. Clicking a resolved link opens the object in the drawer.

An annotation with `type: text` shows the link syntax literally, because that type applies no Markdown.

To write the link, edit the annotation's Value field in the drawer and use the picker. The canvas bubble is read-only.

A PNG export shows the link text with no link styling.

## Limits

- A link does not follow an id rename. After you change an id, every link to the old id renders as broken until you edit it.
- Metron renders a `sightline:` link as plain text with no link styling. It does nothing on click, and its editors show no picker.

## Tetra

A Tetra link targets a zone, group, device, network, information item, system, channel, flow or annotation.

```markdown
Fed from [the historian](sightline:d-hist-1).
Isolated by [the control zone](sightline:z-control).
See also [the commissioning note](sightline:ann-comm).
```

An information item uses its composite id, such as `d-plc-1.i-func`.

| Object | Where the link resolves |
|---|---|
| Zone, group, device, network | The containment tree, at any depth |
| Information item | The device that owns it |
| System | The `systems` list |
| Channel, flow | The `channels` and `flows` lists |
| Annotation | The `annotations` list |

The picker inserts a link such as `[PLC 1](sightline:d-plc-1)`. It lists objects in the order of the table above. A blank or multi-line selection is replaced with the object's label, or with its id when the object has no label.

The picker also leaves out a synthetic channel and an orphans bucket, which the loader creates and you did not write.

Tetra ids are unique within a type, not across types, so a device and a flow can share an id. A link to that id renders as broken. Rename one of the two ids to link to either object.

The description of an existing object and an annotation's Value are the common cases for a link in the props drawer.

A create form is different. Its Description field, and an annotation's Text field, are plain text with no picker. They accept the link syntax, and the link renders once you create the object and open it in the drawer.

## Lamina

A Lamina link targets a cause, event, outcome or control.

```markdown
Driven by [the unpatched host](sightline:C001).
Stopped by [the patch policy](sightline:K001).
Escalates to [the outage outcome](sightline:O002).
```

A control id needs no parent, because control ids are unique across the model. A space in an id looks like `sightline:ic%20cause`.

The picker inserts a link such as `[Cause One](sightline:C001)`. The list shows causes, then events, outcomes and controls. To link to either of two objects that share an id, rename one of the two ids.

The `description` field on the cause, event, outcome and control forms is the common case for a link.

Clicking a resolved link in an annotation bubble does not select the bubble, so the drawer shows the linked object rather than the annotation. Clicking the bubble's other text selects the annotation as before.
