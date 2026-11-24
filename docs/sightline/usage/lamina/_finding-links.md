A rule in a Tetra rule package creates a record for a channel or flow of a Tetra model. A cause in Lamina links that record in its **Links** and pins what it last accepted on the link. Lamina then reports when the record changes.

The props drawer of a cause has an **Open in Tetra** button for each link. See [Jump between a cause and its Tetra channel or flow](/docs/sightline/usage/shared/finding-jump).

## Linked Cause Contents

A linked cause holds the link to the record and, after the first Accept, the pin and an approved snapshot.

| Part | Content |
|---|---|
| Link | The id of the record, the id of the Tetra model and the title of the record when the link was made. A link to a ticket or a URL can sit beside it. |
| Pin | A hash of the data that the rule computed, a hash of the entities that the record depends on, and the data itself. Lamina records the pin at the last Accept. |
| Snapshot | The values that the linked records supplied at the last Accept: the label, the description and the tags. |

The data is the answer that the rule computed. The entities are the channels, flows and zones that the record depends on. The two hashes are separate, so Lamina can tell a new answer from a changed dependency. The pin moves only when a person accepts.

## The States

Lamina compares the pin on the link with the record now. The props drawer shows the state as a short status.

| State | Shown as | Meaning | Action |
|---|---|---|---|
| Current | OK | Both hashes match the pin. | None. |
| Data changed | Moved | The rule computed different data. | Accept |
| Source changed | Moved | An entity that the record depends on changed, and the data did not. | Accept |
| Data and source changed | Moved | Both changed. | Accept |
| Not accepted yet | New | The link holds no pin. | Accept |
| Closed | Closed | The rule no longer creates the record. | Remove link |
| Not found | Missing | No record answers to the id. The row shows the name that the link holds. | Remove link |
| Cannot be read now | Missing | The register of the record cannot be read. | None. Open the drawer again once the register is readable. |
| Not saved yet | Unsaved | The link exists only on the staged cause. | Remove link |
| Record state unavailable | Unknown | The read of the saved record states failed. | Remove link |

When several states apply, the row shows the first in this order: Cannot be read now, Not found, Closed, Not accepted yet. The change states come last. A change that is reverted returns the link to Current.

## Work with the Links in the Props Drawer

The props drawer of a cause shows a **Records** section, a table with one row for each link. The columns are the ids of the flows or channels that the record covers, the name of the record, the status and the row buttons. A section with a count in its heading collapses and expands like the other sections of the drawer.

The status carries the detail. Hover over it, or move the keyboard focus to it, to read the full reason. The **Details** button on the row shows the same text under the row, which also works without a pointer. A changed link states how many data fields changed, for example `1 field changed`, and lists any links of the record that moved. Select the name of the record to see the changed fields in the record drawer, as the old and the new value side by side. The comparison comes from the data that the pin holds, or from the journal of the record when the pin holds none. Without either, the row shows the status alone.

A link that exists only on the staged cause has no saved record state to read. This holds for the link on a cause that **Create cause** has just made. Its row names the record and reads Unsaved, with "Not saved yet" in the detail, until the user selects **Save all**. The row offers **Open in Tetra** and **Remove link**, and no **Accept**.

A list of more than 20 records shows the first 20 and a **Show more** button. The **Add record** and **Withdraw approval** buttons sit in the header of the section.

- **Accept** pins the link at the current hashes and data. It also copies the label, description and tags that the record supplies into the snapshot of the cause. It acts on one link and leaves the pins of the other links alone.
- **Withdraw approval** removes the snapshot and keeps every pin. While a snapshot exists the header shows this button, and an **Approved values** disclosure under the table lists the values.
- **Remove link** deletes the link. It changes no other field.
- **Open in Tetra** opens the Tetra model at the first channel or flow that the record depends on.

Accept, Withdraw approval and Remove link each stage a change to the cause. Accept stages two changes, one to the links and one to the snapshot, so it takes two undo steps. Nothing is written until the user selects **Save all**. The row returns to OK as soon as the change is staged. When editing is not allowed, the buttons are disabled.

The **Links** field of the cause shows no row for a record link. Editing a ticket link in that field leaves every record link and its pin as they were.

## Add a Record to a Cause

The **Records** section of a cause has an **Add record** button. It shows on a cause that links no record yet, so the cause can take its first. The button opens a menu of the records that the cause does not link. The menu lists the Tetra records and Metron findings in the scope of the Findings view, including records that another cause links. Choose a record to add its link, pinned at the state of the record now. The link reads "Not saved yet" until the user selects **Save all**. Adding a record changes no approved value of the cause. When editing is not allowed, the button is disabled.

A record that would break the cause stays in the menu, disabled, and its tooltip says why:

- A Metron finding on a cause that links Tetra records, and a Tetra record on a cause that links findings
- A flow with another source zone, target zone or path zone than the flows of the cause
- A record without zones, when the cause already links flows

## Tetra-Linked Causes

A cause is Tetra-linked when its links hold a Tetra record that describes a flow. The accepted data of that record must hold `sourceZone`, `targetZone` and `pathZones`. A link to a finding of Metron, or to a record without those fields, leaves the cause as it was.

Lamina keeps a Tetra-linked cause consistent with two rules:

- Every flow on the cause has the same source zone, target zone and path zones. The flows can pass through different devices.
- Every event that the cause links has one zone link, and that zone is the target zone of the flows.

A record for a chain of flows in series is a flow record whose data also lists `pivots`, the devices where one chain hop ends and the next starts. A cause that links a chain takes its zones from the whole chain, and its label names the pivots, as in `Remote access then engineering from IT via workstation`. When the record also lists `pivotZones`, the cause line shows each pivot as a chip on the box of its zone. A pivot in the source zone or the target zone is named in the label only, because those boxes wrap the cause and the event. A cause that links two chains needs both to have the same source zone, target zone and path zones. A change to the flows of a chain gives a new record, and the earlier record reads Closed.

The zones come from the data pinned at the last Accept, not from the Tetra model now. A change in Tetra shows as Changed on the record, and Accept then checks the new zones against the cause.

Lamina checks the rules when an edit would break them. The edit is not staged, and a message at the top of the window names the flows or events that break the rule.

| Edit | Rule checked |
|---|---|
| Accept on a record link | The flows, then the events, against the data that the pin would hold |
| Add a record in the **Records** section | The flows, then the events |
| Link an event in **Connected Events** or in **Referenced by Causes** | The events |
| Change or remove the zone link of an event that a Tetra-linked cause links | The events |

Removing a flow, or unlinking an event, is never refused. A cause that links no flow is not subject to either rule.

The bow tie draws the zones of a Tetra-linked cause as boxes along its line, with an empty box for each zone the flow crosses. The page on [authoring controls](/docs/sightline/usage/lamina/authoring-controls#zones-of-a-tetra-linked-cause) describes the boxes. The label of a cause without one of its own lists the names of its flows and ends in `from <source zone>`. The likelihood mode `exposure-grouped` groups a Tetra-linked cause by its source zone.

When the events that break the rule have no zone, the message offers a **Set zone to** button. It names the target zone. Select it to give each of those events that zone and then make the edit. An event that sits in another zone has no such button; change its zone, or link another event.

## Choose the Scope of the Inbox

The scope of the records that the Findings view lists has two settings: the Tetra model to list, and the Tetra zones in scope. The **Model Properties** drawer edits both. Open it in one of two ways:

1. Select **Model** in the toolbar.
2. Right-click an empty part of the canvas or of the Tabular Editor, then choose **Edit…**.

Choose a Tetra model, then add the zones in scope from the **Zones in scope** list. Type in its field to filter the zones, and choose one to add it as a row. The list offers the zones that the chosen model defines. While no model is chosen, it offers the zones of every Tetra model in the workspace, in any folder.

Save the model to keep the scope. A zone that the chosen model does not define stays as a row, with the note "not defined in this Tetra model". Select **×** on its row to remove it. Choose **(any model)** and remove every zone to remove the scope.

Without a chosen model, the dialog lists the records of every Tetra model that the workspace holds.

## List the Records That No Cause Links

Run **Lamina: Open Findings Inbox** from the command palette to show the Findings view. Below the findings of Metron, a **Tetra records** list shows the records in scope that no cause of the open model links. Records from a rule that hides its output do not appear, and a closed record does not appear.

Select **Create cause** on a row to open the new cause drawer. The drawer links the record, pins its current state and approves it into the snapshot. The row leaves the list when you create the cause.

The list can show a message in place of rows:

| Message | Cause |
|---|---|
| A request to declare zones | No zones are in scope. Select zones in the **Model Properties** drawer. |
| Tetra model example-ot-model is not in this workspace. | The chosen Tetra model is not in the workspace. Choose another model in the **Model Properties** drawer. |
| Every Tetra record in scope has a cause. | Every record in scope has a link. |

## Problems Panel

The Problems panel shows a warning for a link in one of five states. These are Data changed, Source changed, Data and source changed, Closed and Not found. The changed warning names the cause, the record and what moved. A link that reads Cannot be read now appears as an error, because Lamina cannot check it. A current link and a link that is not accepted yet give no entry. A deployment that refuses the read gives no entry either.

## Model Scope of Finding-Link Reads

The Problems-panel warnings, the **Model Properties** choices and the **Records** section cover a model that meets both conditions:

- The model sits in the first workspace folder.
- No other model in the workspace has the same id.

For any other model, the Problems panel shows no record warnings. The **Model Properties** drawer shows "Could not list Tetra models" in place of the choices. Each block in the **Records** section reads "Record state unavailable", and offers **Open in Tetra** and **Remove link**.

## Models from an Earlier Release

Some links from earlier releases are not read, and the props drawer and the Problems panel report nothing for them. Remove such a link and link a record instead.

A model saved with the earlier finding links fails to load until a migration tool converts it. Ask Corvis for the tool that matches your release. A migrated link reads as changed until a person selects **Accept**.
