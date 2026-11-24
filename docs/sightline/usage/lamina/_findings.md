Lamina lists the Metron findings and the Tetra records in a model's scope, and creates a cause that links one or more of them. It never writes a finding or a record. A cause stores the link, and Lamina shows the state of each linked finding or record on the cause.

## The Findings Inbox

Run **Lamina: Open Findings Inbox** from the command palette. The **Findings** view shows under the Sightline icon, below the Elements view. It starts collapsed to its header, so it takes one row of the sidebar until you open it, and **Lamina: Open Findings Inbox** expands it and moves focus to it. VS Code then remembers whether you left it open or collapsed. It shows while a Lamina model is the one you last focused. In the web version of Sightline, the same tree shows on the **Findings** tab of the bottom panel. Opening the Findings Inbox shows the panel on that tab. The tab lists the findings of the active Lamina tab. With no tab open it reads "Open a model to see its findings." The view and the tab show one tree with two groups. Above the tree, **Findings table** opens the same records as a table, as [The Findings table](#the-findings-table) describes.

- **Metron findings.** The finding records of the Tetra model that the Lamina model names in its scope, grouped by rule. When the scope names no model, Lamina lists the findings of every Tetra model in the workspace. A Metron finding needs no zone, so this group shows even when the scope holds no zones.
- **Tetra records.** The records of that Tetra model in the zones of the scope, grouped by zone and then by rule. A zone group shows the label of the zone in the Tetra model, and the zone id when the zone has no label. Hover the group to see the id. Set the scope in the **Model Properties** drawer. When the scope holds no zones, the group is missing and a note tells you to choose the zones in scope in that drawer. See [Record links](/docs/sightline/usage/lamina/finding-links).

A Tetra record is placed in its zone by the `zoneId` part of its key. A record with no `zoneId` part, or with a `zoneId` outside the scope, is not listed, and a note above the tree gives the count.

Each group shows the number of records under it, and its arrow collapses or expands it. The filter box above the tree narrows the rows to the records whose name or mark contains the text. A record row shows the key values of the record, such as an asset or a target. After the name, a row shows the state of the record: **Current**, **Data changed**, **Source changed** or **Data and source changed**. **has a cause** follows the state when a cause of the open model links the record. The groups above it already name the zone and the rule. When the data of a Tetra record has a `label` field with a non-empty text, the row shows that label instead of the key values. Hover a row to see the record title and the data of the record, or the label, the key and the data when the record has a `label`, such as the `pathway`, `sourceZone` and `flowType` of a Tetra record. The data helps you choose the cause to create. A Metron finding also shows its description there.

A record that a cause of the open model links stays in the tree and reads **has a cause**, so the answer is per model. A record that its rule closes does not appear. A finding appears in the Metron group only when its rule does not hide it.

A note above the tree says why a group is missing:

- No Metron findings are in scope
- No zones are in scope
- No Tetra records are in scope
- The Tetra model is not in the workspace
- Records are left out, for example "12 records from 2 rules name no zone in scope" or "1 record from 1 rule names a zone outside the scope". One note appears for each reason

Findings that someone writes by hand outside Metron are not read. See [Findings in Metron](/docs/sightline/usage/metron/findings).

In a window that cannot edit the model, the view lists the records and **Create cause** stays disabled.

## Creating a Cause from a Finding

Click a Metron finding or a Tetra record to pick it, then press **Create cause** in the toolbar of the view. The right-click menu of the row does the same. A double-click on a row opens the record instead, as [Opening a record](#opening-a-record) describes. **Create cause** stays disabled on a record that already has a cause. The props drawer opens on its new-cause surface, and the view stays open. The cause is a draft, prefilled with the finding's title as its label (the `label` of a Tetra record that has one) and, for a Metron finding, the finding's description. Adjust any field, pick the destination file, and press **Create**.

To make one cause from several records, Ctrl-click (Cmd-click on a Mac) or Shift-click more records, then press **Create cause**. The cause links every picked record, in the order that you picked them, and takes its label and description from the first. A pick holds Tetra records or Metron findings, not both. Tetra records in one pick must share their source zone, target zone and path zones. When a pick breaks a rule, **Create cause** stays disabled and its tooltip names the reason.

The prefill carries a record link for each picked record, pinned at the state of the record now. A link to a Tetra record takes the title of the record's rule as its name, such as "Data flow into zone", so the **Vector** of the cause lists the rule once, even when several of its records come from the same rule. A record whose rule is no longer in its rule package keeps the record title as the link name. A cause that exists already keeps the link names it was created with. The prefill also carries a snapshot of the label, description and tags that the finding supplies. The cause starts **Current** and approved. **Create** stages the cause with the link in the pool of unsaved changes, and the next **Save all** commits both together. The record reads **has a cause** as soon as the cause is staged. Nothing is written to any other file. **Cancel** closes the drawer and leaves the finding unmarked.

When a Risk is open on the Diagram, **Create** also adds the new cause to that Risk. The cause and the change to the Risk wait in the pending changes together, and **Save all** writes both. The cause draws on the canvas at once and shows as an included element in the Elements view. The rating of the Risk counts the cause after **Save all**. With no Risk open, **Create** adds the cause alone. When the Elements view cannot change the Risk, the cause is still created and the view gives the reason.

## Opening a Record

Double-click a Metron finding or a Tetra record, or select it and press Enter, to open it in the record drawer. **Open record** in the row menu does the same. The drawer opens on the Diagram, beside the properties drawer when one is open, and the view stays open. It works in the web version of Sightline too, and in a window that cannot edit the model.

The drawer is read-only. It shows the title and the state of the record, its rule, package and model, the parts of its key, and its data. The data starts with the label, the tags and the lists of ids, then shows the whole payload as a tree. An id that names an element of the open model is a link. The drawer also lists the causes of the open model that link the record. Choose a cause to select it on the Diagram. A Tetra record has an **Open in Tetra** button, which works in the VS Code extension only. The drawer shows no journal and no changes since a cause accepted the record. Open the record from the cause drawer to see those. See [Record links](/docs/sightline/usage/lamina/finding-links).

Opening a record changes nothing and stages nothing. Choose another record to replace the drawer, or press **Close** to close it. The drawer also closes when a reload no longer lists the record.

## The Row Menu

Right-click a record to open its menu. Create cause stays at the foot of the menu, below a rule. The menu above it holds these entries. **Open record** comes first and shows in bold, because it is the action of a double-click.

| Entry | What it does | Disabled when |
|---|---|---|
| **Open record** | Opens the record in the record drawer. See [Opening a record](#opening-a-record). | Never. |
| **Add to cause** (named for the selected cause) | Adds the picked records to the cause that is selected on the Diagram. The cause keeps its other links, and the change waits in the pending changes until **Save all**. | The window cannot edit the model, no cause is selected, or the cause already links the record. A pick that breaks the cause, such as records of two sources, adds nothing and the panel reports the reason. |
| **Open in Tetra** | Opens the Tetra model at the channel or flow that the record refers to. | Shown for Tetra records in the VS Code extension only. |
| **Show linked causes** | Lists the causes of the open model that link the record. Choosing one selects it on the Diagram. | No cause links the record. |
| **Copy record id** | Copies the id of the record to the clipboard. | Never. |

**Add to cause** and **Show linked causes** work in the web version of Sightline too. The web version has no **Open in Tetra** entry.

## The Findings Table

Press **Findings table** above the tree to see the records as a table. The table replaces the Diagram in the same editor, and the tree stays open for quick picking. It works in the web version of Sightline too, and in a window that cannot edit the model.

The table has one row for each Metron finding and Tetra record in the scope:

| Column | What it shows |
|---|---|
| **Record** | The name of the record, as the tree shows it |
| **Source** | **Tetra** or **Metron** |
| **Rule** | The title of the rule, or the rule id when the rule package lists no title |
| **Zone** | The zone label, or the zone id when the zone has no label. A Metron finding has no zone |
| **State** | **Current**, or **Changed** when the data or the source of the record changed |
| **Linked causes** | The labels of the causes of the open model that link the record |
| **Summary** | The first three fields of the record data, or the description of a Metron finding |

The notes about records the scope leaves out show above the table, as they do above the tree.

Use **Group** to group the rows by zone, rule, state or source. **Filter** narrows the rows by any column, the search box narrows them by text, and **Columns** hides or reorders the columns. A double-click on a record, or Enter on a selected record, opens it in the record drawer. See [Opening a record](#opening-a-record).

Tick the boxes of one or more records, then press **Create cause**. The cause links every ticked record, in the order that you ticked them, and the drawer opens as it does from the tree. The rules for a pick are the same. A pick holds Tetra records or Metron findings, not both. Tetra records in one pick must share their source zone, target zone and path zones. **Create cause** stays disabled in four cases:

- Nothing is ticked
- A ticked record already has a cause
- The pick breaks a rule
- The window cannot edit the model

The tooltip of the button names the reason. A record that leaves the scope on a reload drops out of the ticked set.

Press **Back to diagram** to return to the Diagram. The segments of the **Diagram** and **Tabular** switch in the toolbar also leave the table. The table keeps its grouping and its ticks until you close the editor.

## The Linked Finding on the Cause

A cause that links a finding shows the finding in its drawer. The state reads **Current**, **Data changed**, **Source changed**, **Data and source changed**, **Not accepted yet**, **Closed**, **Not found** or **Cannot be read now**. See [Record links](/docs/sightline/usage/lamina/finding-links) for **Accept**, **Withdraw approval**, **Remove link** and the explanation of what changed.

A finding link is stored once, in the cause's own record. A finding that a cause links is listed by Metron too, under the finding's usages. For how the apps share findings, see the [Cross-app workflows](/docs/sightline/architecture/cross-app-workflows) page.
