A rule in a Tetra rule package creates a record for a channel or flow of a Tetra model. A cause in Lamina links that record from its vector and pins what it last accepted on the link. Lamina then reports when the record changes.

The props drawer of a cause has an **Open in Tetra** button for each link. See [Jump between a cause and its Tetra channel or flow](/docs/sightline/usage/shared/finding-jump).

## What a linked vector holds

A linked vector holds the link to the record and, after the first Accept, the pin and an approved snapshot.

| Part | Content |
|---|---|
| Link | The id of the record, the id of the Tetra model and the title of the record when the link was made. A link to a ticket or a URL can sit beside it. |
| Pin | A hash of the data that the rule computed, a hash of the entities that the record depends on, and the data itself. Lamina records the pin at the last Accept. |
| Snapshot | The values that the linked records supplied at the last Accept: the label, the description and the tags. |

The data is the answer that the rule computed. The entities are the channels, flows and zones that the record depends on. The two hashes are separate, so Lamina can tell a new answer from a changed dependency. The pin moves only when a person accepts.

## The states

Lamina compares the pin on the link with the record now.

| State | Meaning | Action |
|---|---|---|
| Current | Both hashes match the pin. | None. |
| Data changed | The rule computed different data. | Accept |
| Source changed | An entity that the record depends on changed, and the data did not. | Accept |
| Data and source changed | Both changed. | Accept |
| Not accepted yet | The link holds no pin. | Accept |
| Closed | The rule no longer creates the record. | Remove link |
| Not found | No record answers to the id. The block shows the name that the link holds. | Remove link |
| Cannot be read now | The register of the record cannot be read. | None. Open the block again once the register is readable. |

When several states apply, the block shows the first in this order: Cannot be read now, Not found, Closed, Not accepted yet. The change states come last. A change that is reverted returns the link to Current.

## Work with the links in the props drawer

The props drawer of a cause shows a **Records** section with one block for each link. A changed link lists what moved, for example `type 'system' -> 'security'`. The list comes from the data that the pin holds, or from the journal of the record when the pin holds none. Without either, the block shows the state alone.

- **Accept** pins the link at the current hashes and data. It also copies the label, description and tags that the record supplies into the snapshot of the vector. It acts on one link and leaves the pins of the other links alone.
- **Withdraw approval** removes the snapshot and keeps every pin. The section shows the approved values and this button while a snapshot exists.
- **Remove link** deletes the link. It changes no other field.
- **Open in Tetra** opens the Tetra model at the first channel or flow that the record depends on.

Accept, Withdraw approval and Remove link each stage one change to the vector of the cause. Nothing is written until the user selects **Save all**. The block returns to Current as soon as the change is staged. When editing is not allowed, the buttons are disabled.

The **Links** field of the vector shows no row for a record link. Editing a ticket link in that field leaves every record link and its pin as they were.

## Choose the scope of the inbox

The scope of the records that the Findings dialog lists has two settings: the Tetra model to list, and the Tetra zones in scope. The **Model Properties** drawer edits both. Open it in one of two ways:

1. Select **Model** in the toolbar.
2. Right-click an empty part of the canvas or of the Tabular Editor, then choose **Edit…**.

Choose a Tetra model, then select the zones in scope. The zone list shows the zones that the chosen model defines. While no model is chosen, it shows the zones of every Tetra model in the workspace, in any folder.

Save the model to keep the scope. A zone that the chosen model does not define stays selected, with the note "not defined in this Tetra model". Clear its checkbox to remove it. Choose **(any model)** and clear every zone to remove the scope.

Without a chosen model, the dialog lists the records of every Tetra model that the workspace holds.

## List the records that no cause links

The **Findings** button in the toolbar opens the Findings dialog. Below the findings of Metron, a **Tetra records** list shows the records in scope that no cause of the open model links. Records from a rule that hides its output do not appear, and a closed record does not appear.

Select **Create cause** on a row to open the new cause drawer. The drawer links the record, pins its current state and approves it into the snapshot. The row leaves the list after the save.

The list can show a message in place of rows:

| Message | Cause |
|---|---|
| A request to declare zones | No zones are in scope. Select zones in the **Model Properties** drawer. |
| Tetra model example-ot-model is not in this workspace. | The chosen Tetra model is not in the workspace. Choose another model in the **Model Properties** drawer. |
| Every Tetra record in scope has a cause. | Every record in scope has a link. |

## Problems panel

The Problems panel shows a warning for a link in one of five states. These are Data changed, Source changed, Data and source changed, Closed and Not found. The changed warning names the cause, the record and what moved. A link that reads Cannot be read now appears as an error, because Lamina cannot check it. A current link and a link that is not accepted yet give no entry. A deployment that refuses the read gives no entry either.

## Which models these reads cover

The Problems-panel warnings, the **Model Properties** choices and the **Records** section cover a model that meets both conditions:

- The model sits in the first workspace folder.
- No other model in the workspace has the same id.

For any other model, the Problems panel shows no record warnings. The **Model Properties** drawer shows "Could not list Tetra models" in place of the choices.

## Models from an earlier release

Some links from earlier releases are not read, and the props drawer and the Problems panel report nothing for them. Remove such a link and link a record instead.

A model saved with the earlier finding links fails to load until a migration tool converts it. Ask Corvis for the tool that matches your release. A migrated link reads as changed until a person selects **Accept**.
