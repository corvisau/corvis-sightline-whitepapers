A cause or an outcome owns its controls. Each control belongs to one parent, and
reuse comes from a template.

This page covers how to add a control, reuse a template, set the elements a
control covers and read where an inherited value came from. For the
effectiveness ratings and the zone rules that colour a control's element badges,
see [Rate a Risk](/docs/sightline/usage/lamina/risks#rate-a-risk).

## Add a Control

A control always belongs to the cause or the outcome that owns it. Two parents
never share one control.

1. Right-click the cause or the outcome on the canvas.
2. Select **Add control**.
3. Fill in the **Label** and the other required fields in the drawer that opens.

A control on a cause is preventative. A control on an outcome is mitigative.

The context menu offers **Add control**, **Add inherent control** and **Add conditional modifier** on the canvas only. See [Inherent controls](#inherent-controls) and [Conditional modifiers](#conditional-modifiers). The Tabular Editor
has no equivalent, and a control row there cannot be duplicated.

When the Tabular Editor lists more than one type, each row starts with an icon
for its type: the Elements view icon for a cause, event or outcome, and a
generic icon for a control. The **type** column still names it. With one type in
the grid, the rows have no icon.

Two relationships are references and not owned objects. A cause points at the
events it can lead to, and an event points at the outcomes it can produce.
Several causes can point at one event, and several events at one outcome.

## Inherent Controls

An inherent control is a baseline control that already exists before any assessed control. You record whether it is valid. A cause or an outcome can own one.

To add an inherent control from the props drawer:

1. Select the cause or the outcome on the canvas to open the props drawer.
2. On a cause, select **+ Add** in the **Inherent Controls** list. On an outcome, select **+ Add inherent control** in the **Inherent and Conditional Controls** list.
3. Select **Edit →** beside the new row.
4. Enter the **Label** and set **Valid**.

To add one from the canvas, right-click the cause or the outcome and select **Add inherent control**. The drawer opens on the new row.

To add an inherent control from a control template, use the **Add control from template** section of the drawer. Choose a template, then select **+ Add**. The list holds every control template of type Inherent, and on an outcome every template of type Conditional as well. It does not depend on the **Suggested controls** of the cause or the outcome, which remain a shortcut to the templates they name.

## Conditional Modifiers

A conditional modifier is a barrier on a consequence line that counts only when a stated condition holds. An example is "applies only if the isolation valve is closed". Only an outcome can own one.

A conditional modifier is one of three kinds of control:

| Kind | Where it sits | What it records |
|---|---|---|
| Assessed control | Cause or outcome | Effectiveness ratings, elements and zones. It always applies. |
| Inherent control | Cause or outcome | A baseline control that is valid or not valid |
| Conditional modifier | Outcome | A free-text condition for when the barrier counts |

To add a conditional modifier from the props drawer:

1. Select the outcome on the canvas to open the props drawer.
2. In the **Inherent and Conditional Controls** list, select **+ Add conditional modifier**.
3. Select **Edit →** beside the new row.
4. Enter the **Label** and the **Condition**.

To add one from the canvas, right-click the outcome and select **Add conditional modifier**. The drawer opens on the new row, ready for the **Label** and the **Condition**.

An outcome can also add a control template of type `Conditional`. Use its **Suggested controls** section or its **Add control from template** section. A cause never offers a conditional modifier, in the drawer, in the template list or in its context menu.

The condition is plain text. Lamina does not evaluate it, and a conditional modifier never changes the residual rating. It has no effectiveness rating, elements or zones.

On the canvas, a conditional modifier has a dotted border, the category **Conditional** and a **Condition** row. It sits between the assessed controls and the inherent controls on the consequence line. Select **Edit** on its context menu, or double-click it, to open its form in the drawer. Dragging reorders it among the conditional modifiers only.

The PowerPoint risk report lists each conditional modifier and its condition under **Conditional Modifiers** in the **Outcomes** table. See [Risk report (PowerPoint)](/docs/sightline/usage/lamina/risk-report-pptx).

## Effectiveness Labels

An assessed control has a fixed set of effectiveness ratings: `Not Assessed`, `None`, `Low`, `Partial`, `Full` and `Not Applicable`. The risk matrix file of the model can give a display label to each rating under `effectiveness`. For example, `Partial: Moderate` shows the rating `Partial` as "Moderate".

The control on the canvas, the **Target Effectiveness** and **Current Effectiveness** fields in the props drawer, the draw.io export and the generated reports show the label. A rating with no label shows its own name. The model file keeps the rating, and the colour of the cell follows the rating.

## Reuse Comes from a Template

A template holds the shared part of a control. Each control that uses it
stores only the fields that differ.

1. Select the control on the canvas to open the props drawer.
2. In the **Template** field, type part of a template name, or select **Browse** to pick from the tree.
3. Set the fields that differ for this control, such as the current effectiveness rating.

The control takes every other value from the template. A field that the control
sets wins over the template's value. Set only the fields that differ, because a
control that repeats the template's own values no longer follows a later change
to the template.

Select **Edit** beside the **Template** field to open that template in the
Template Editor.

### Promote a Control to a Template

To reuse a control you have already built, promote it. Promote works the same on
a cause, an event and an outcome.

1. Right-click the control on the canvas and select **Promote to template…**. In the Tabular Editor, right-click a cause, event or outcome row. A row for a control has no such entry, so promote a control from the canvas.
2. Check the template id and the label. The id starts with `tpl-` and the template type, and must not match an existing template.
3. Choose the template file. The list shows the template files that the model declares. When the model declares none, enter a name for a new file, such as `templates.yaml`.
4. Untick any field you do not want in the template. The dialog offers the fields that a template can hold. It leaves out the identity, links, ratings and zones of this control, because those belong to this control alone.
5. Select **Create template**.

The control itself does not change. It does not start to use the new template. To
make it use the template, set the **Template** field on the control. The new
template appears in the **Template** field list after the model reloads.

An inherent or conditional control has no **Promote to template…** entry. A model
that you can only view shows the entry greyed out.

## Elements: The IT/OT Elements a Control Covers

A control's **Elements** field names the assets it protects. Each row holds an
element id and an optional zone narrow.

- A row with no zone covers that element in every zone.
- A row with one zone covers the element in that zone only.
- A row with several zones covers the element in those zones only.

Select **+ Add element** to append a row, then type the element id and add any
zones. Select **×** to remove a row.

A row can carry extra detail, such as a name and a description. The drawer keeps
that detail when you edit the row, but it shows only the id and the zones.

### Zone IDs Name Tetra Zones

A zone id on an outcome, objective, control or element narrow names a zone in
your Tetra model. Lamina accepts any
non-empty string and has no zone of its own.

Lamina checks zone ids when it opens a model, and again when you save a file in
the workspace. It adds a Problems-panel warning to the model for each zone id
that no Tetra model defines.

The check reads every Tetra model that the workspace lists, in any folder. A
Tetra model needs an id to be listed. The check gives no warning for a model
that the workspace does not list.

## Zone Control Groups

A zone control group is the set of preventative controls on one cause line
that sit in one Tetra zone. The bow tie draws a labelled box around each group,
so you can see which zone each control protects. The box is labelled with the
zone's name, or with its id when the link has no name.

To put a control in a group, add a link to the control's **Links** field:

1. Set the relationship to `zone`.
2. Set the target kind to `entity`.
3. Choose the zone from the target list, which also sets the instance to its
   Tetra model, or type the Tetra zone id.
4. Optionally, give the link a name, such as `Remote access DMZ`. The box shows
   that name.

Boxes appear on a cause line only when at least one of its assessed controls
has a zone link. A line with no zone links draws as it always has. On a line
with boxes, each zone's controls sit together. The zones appear in the order
their first control appears in the control list.

A control goes in the line's **Invalid zone** box when its zone link:

- is missing
- names more than one zone
- names a zone that no Tetra model in the workspace declares

The Invalid zone box has a dashed border. Hover over its label to see each
control in it and why it is there. Inherent controls and conditional modifiers
are never grouped.

You can drag a control to reorder it inside its own box. To move a control to
another zone, edit its zone link. Dragging it into another box does not
change its zone.

The draw.io export and the PNG export both include the boxes.

The zone link is separate from the control's **Zones** field. The **Zones**
field picks which zones' assessments apply to the control and can name several
zones. A mitigative control with none takes the zones of its outcome, and a
preventative control with none applies to no zone. The zone link names the one
zone the control sits in, and a control without one has no zone.

### Source and Target Zones

An event can carry a zone link. It names the zone the event targets, and the
bow tie draws a box around the event labelled `Target: <zone>`. A cause has no
zone link. A cause that links a Tetra flow takes its source zone from the flow,
and the bow tie draws a box around it labelled `Source: <zone>`. The zone
control groups sit between the two boxes, so such a cause line shows its whole
path: the source zone, the zones whose controls it crosses, and the target zone.

An event without a zone link has no box. A zone link that names more than one
zone, or a zone that no Tetra model declares, makes the box read `Target:
Invalid zone` and gives it a dashed border. Hover over the label to see why.

An event shared by several causes has one box. Lines from causes in different
source zones all end in that one Target box.

### Zones of a Tetra-Linked Cause

A cause that links a Tetra record for a flow takes its zones from that
flow, not from a zone link. The bow tie draws them along the cause line, from
the cause to the event:

- A `Source: <zone>` box around the cause, from the source zone of the flow.
- One box for each zone the flow crosses between its source and its target, in
  path order.
- A `Target: <zone>` box around the event.

Each box shows the zone id. An event with a zone link of its own keeps that
link in its Target box. An event without one takes the target zone of the flow.

A path box is drawn even when no control sits in that zone. An empty box has a
dotted border and leaves room for controls. A control goes in a path box when
its zone link names that zone.

The control boxes follow the flow, from the cause to the event:

1. The source zone box, when a control names the source zone.
2. The path boxes, in path order.
3. The target zone box, when a control names the target zone.
4. A box for each other zone, in the order its first control appears.
5. The Invalid zone box.

The source and target boxes are not drawn empty, because the Source and Target
boxes already mark those zones. A zone the flow leaves and enters again keeps
one box, at its first place. The controls inside a box keep their list order,
and the order of the boxes changes only the drawing, not the file.

A cause that links no flow draws no Source box and no path boxes. Its control
boxes run in the order the first control of each zone appears, except that the
box for the zone of the cause's events comes last, before the Invalid zone box.

## A Template Can Derive from Another Template

Reuse is hierarchical: define the shared part once, then derive the specific
case from it. A control that uses the specific template takes the shared values
from the generic template and the specific values from the specific one. Each
level overrides the one above it, and a chain can be any depth.

In the Template Editor, select **+ New template** to create a template. The
create drawer has an optional **Template** selector under the label field. Pick
the parent there to create a sub-template, or leave it empty for a top-level
template. The selector lists the templates of the chosen type, and changing the
type clears a parent of the old type.

To re-parent a template, open its row and change the **Template** selector in
the drawer. Clear it to make the template top-level.

The Template Editor refuses a save that would make a template derive from
itself. A template cannot be its own parent or the child of one of its own
sub-templates. The save bar shows the reason, and nothing is saved.

## Sources of an Inherited Value

The props drawer marks an inherited field with an **inherited** badge. Hover the
row to see which templates the value came through:

```
Generic EDR control -> EDR product A used in Zone X -> this control
```

The chain reads left to right, from the template that introduced the value to
the control in front of you. A field that the nearest template overrides names
only that template.

The picker's **Browse** button shows the same hierarchy as a tree, with each
sub-template under its parent. Typing in the field filters the tree and keeps a
match's parents visible.

A template that derives from a template of a different type appears in the
picker as a top-level entry with no warning. An example is a control template
that names a cause template. Check the parent's type when a sub-template does
not appear where you expect it.

## Open the Template Editor and the Constants Editor

The diagram toolbar has a **Templates** button and a **Constants** button. In VS Code each opens an editor panel beside the diagram. Both buttons are disabled in a read-only view, because both editors write to the model. The **Lamina: Edit Templates** and **Lamina: Edit Constants** commands open each editor in a read-only view too, with every control disabled.

The Template Editor lists the templates in a grid. With **All types** selected the grid shows the id, label, type and source file. When you select one type, the grid shows the fields of that type after the id and the label, and the source file last. A cell for a text, number, true/false or choice field edits in place: click the cell, change the value and press **Enter**. The edit stays pending until you select **Save all**. A field with its own editor, such as the template parent, opens in the drawer when you click the row outside an editable cell. The **Columns**, **Filter** and **Group** controls in the grid header work as they do in the Tabular Editor. With one file and one type selected, **Move up** and **Move down** on a row change the order of the templates in the file.

The Constants Editor lists every template file in the model, adds and deletes constants, and keeps the type of each value. Its [Tetra page](/docs/sightline/usage/tetra/constants-editor) describes these controls, and the Lamina editor works the same way.

In the web shell each button opens its editor in a tab of its own, named after the model. Select the button again to return to the tab that is open. The editor saves to the model, and the diagram tab shows the saved change. A Lamina page opened outside the web shell has no shell to open a tab in, so it shows neither button.

### Zone Box Colours

A zone box takes the colour of its zone in the Tetra model. Tetra colours a zone
by its zone type, using the colour that the Tetra manifest defines for that type.
A zone whose type has no colour takes the colour of its own style. The box has a
3 px border in that colour and a light tint of it as the fill. The tint follows
the light and dark themes.

A zone with no colour in Tetra has a neutral border of the same width and no
fill. The Invalid zone box stays dashed amber and has no fill. An empty box for
a zone that no control names is dotted, in the colour of its zone.

When two Tetra models declare the same zone id, the box takes its colour from
the first of these that declares the zone:

1. The Tetra model named as the instance of the zone link. A box with no link of
   its own uses the model of the cause's flow.
2. The Tetra model set in the Model Properties drawer.
3. The Tetra model whose id sorts first.

The draw.io export and the PNG export draw the same colours. The `lamina-png`
command finds Tetra models in the folder of the manifest and its subfolders only.
A model in another folder draws neutral boxes in a PNG from that command.
