A cause or an outcome owns its controls. Each control belongs to one parent, and
reuse comes from a template.

This page covers how to add a control, reuse a template, set the elements a
control covers and read where an inherited value came from. For the
effectiveness ratings and the zone rules that colour a control's element badges,
see [Rate a Risk](/docs/sightline/usage/lamina/risks#rate-a-risk).

## Add a control

A control always belongs to the cause or the outcome that owns it. Two parents
never share one control.

1. Right-click the cause or the outcome on the canvas.
2. Select **Add control**.
3. Fill in the **Label** and the other required fields in the drawer that opens.

A control on a cause is preventative. A control on an outcome is mitigative.

The context menu offers **Add control** on the canvas only. The Tabular Editor
has no equivalent, and a control row there cannot be duplicated.

Two relationships are references and not owned objects. A cause points at the
events it can lead to, and an event points at the outcomes it can produce.
Several causes can point at one event, and several events at one outcome.

## Conditional modifiers

A conditional modifier is a barrier on a consequence line that counts only when a stated condition holds. An example is "applies only if the isolation valve is closed". Only an outcome can own one.

A conditional modifier is one of three kinds of control:

| Kind | Where it sits | What it records |
|---|---|---|
| Assessed control | Cause or outcome | Effectiveness ratings, elements and zones. It always applies. |
| Inherent control | Cause or outcome | A baseline control that is valid or not valid |
| Conditional modifier | Outcome | A free-text condition for when the barrier counts |

To add a conditional modifier:

1. Select the outcome on the canvas to open the props drawer.
2. In the **Inherent and Conditional Controls** list, select **+ Add**.
3. Select **Edit →** beside the new row.
4. Enter the **Label** and the **Condition**.

A control template whose control type is `Conditional` can also be added from the **Suggested controls** section of an outcome. A cause never offers one.

The condition is plain text. Lamina does not evaluate it, and a conditional modifier never changes the residual rating. It has no effectiveness rating, elements or zones.

On the canvas, a conditional modifier has a dotted border, the category **Conditional** and a **Condition** row. It sits between the assessed controls and the inherent controls on the consequence line. Select **Edit** on its context menu, or double-click it, to open its form in the drawer. Dragging reorders it among the conditional modifiers only.

The PowerPoint risk report lists each conditional modifier and its condition under **Conditional Modifiers** in the **Outcomes** table. See [Risk report (PowerPoint)](/docs/sightline/usage/lamina/risk-report-pptx).

## Reuse comes from a template

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

## Elements: the IT/OT elements a control covers

A control's **Elements** field names the assets it protects. Each row holds an
element id and an optional zone narrow.

- A row with no zone covers that element in every zone.
- A row with one zone covers the element in that zone only.
- A row with several zones covers the element in those zones only.

Select **+ Add element** to append a row, then type the element id and add any
zones. Select **×** to remove a row.

A row can carry extra detail, such as a name and a description. The drawer keeps
that detail when you edit the row, but it shows only the id and the zones.

### Zone ids name Tetra zones

A zone id on a cause, outcome, exposure, objective, control, element narrow or
vector source zone names a zone in your Tetra model. Lamina accepts any
non-empty string and has no zone of its own.

Lamina checks zone ids when it opens a model, and again when you save a file in
the workspace. It adds a Problems-panel warning to the model for each zone id
that no Tetra model defines.

The check reads every Tetra model that the workspace lists, in any folder. A
Tetra model needs an id to be listed. The check gives no warning for a model
that the workspace does not list.

## A template can derive from another template

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

## Where an inherited value came from

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

In the web shell each button opens its editor in a tab of its own, named after the model. Select the button again to return to the tab that is open. The editor saves to the model, and the diagram tab shows the saved change. A Lamina page opened outside the web shell has no shell to open a tab in, so it shows neither button.
