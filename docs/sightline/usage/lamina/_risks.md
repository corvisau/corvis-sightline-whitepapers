A Risk groups cause-event-outcome paths of a bow-tie diagram. A model manifest can define Risks, and a model that defines none has none. This page covers what a Risk groups and how to define one. It also covers how a Risk selects its members, how a Risk is rated, and what the read returns.

## Paths That a Risk Groups

A path runs from a cause through an event to an outcome. A cause lists the events it can lead to, and an event lists its outcomes. Each combination of a cause, one of its events and one outcome of that event is one path.

A Risk counts its members: the causes, events and outcomes that it includes, that its program selects, or that a Risk it holds counts. It groups every path through a member event, one of its member causes and one of its member outcomes.

A path can sit in more than one Risk.

## Define Risks in the Manifest

Add a `risks` list to the `*.bowtie.yaml` manifest. This example defines two Risks:

```yaml
risks:
  - id: R001
    label: Station outage
    include:
      - id: C-001
      - id: E-001
      - id: O-001
  - id: R002
    label: Power events
    include: []
    filter:
      program:
        - model |> where("item.kind == \"event\" && item.tags |> hasAny(\"power\")") |> add()
```

| Field | Content |
|---|---|
| `id` | Required. Starts with a letter, then letters, digits, `.`, `_` or `-`. Each Risk in the model has its own id. |
| `label` | The display text of the Risk, as for every other entity. A manifest of schema version 4 or earlier wrote it as `name`. |
| `description` | A note about the Risk. |
| `include` | Required. The elements the Risk counts, one `id` for each. An empty list is a Risk with no members of its own. See [Risk membership sources](#risk-membership-sources). |
| `exclude`, `filter`, `risks`, `flattenSubRisks` | The elements the Risk never counts, its program, the Risks it holds, and how a held Risk counts. See [Risk membership sources](#risk-membership-sources). |
| `folder` | The folder the Risk sits in. See [Folders](#folders). |
| `aggregation` | How the Risk aggregates. See [Set how a Risk aggregates](#set-how-a-risk-aggregates). |
| `viewOptions` | The saved **View** menu settings of the Risk. **Save view** writes it. See [Save and revert the view](#save-and-revert-the-view). |

A Risk with no `include` key, or with any other key, is invalid.

A manifest of schema version 4 or earlier wrote the label of a Risk as `name`. Lamina reads such a manifest with `label` in its place when it opens the model, and the next save writes `label`. The web shell and the API refuse an edit that still holds `name` on a Risk until the model folder is migrated. The [compatibility page](/docs/sightline/upgrade/compatibility) describes the command that rewrites the files.

A manifest of schema version 3 or earlier listed Risks as `parts` and named data slices. Lamina reads each data slice as a Risk with a program and each Risk of parts as a Risk that includes its elements when it opens the model, and refuses an edit until the model folder is migrated. The [compatibility page](/docs/sightline/upgrade/compatibility) describes the command that rewrites the files.

A manifest that defines Risks has exactly those Risks. The read lists the events that no Risk includes.

A build of Sightline from before this release refuses to load a manifest that has a `risks` list, a `riskDefaults` key or a Risk with `aggregation`.

## Risk Membership Sources

A Risk takes its members from three sources: the elements it includes by id, the elements its program selects, and the elements of the Risks it holds. The [Elements view](#elements-view) includes and excludes elements. The [Risk drawer](#risk-drawer) sets the program and the held Risks. The Model Tree lists every Risk.

A member is a cause, an event or an outcome. A Risk counts its members and nothing else. A path runs through a member event, one of its member causes and one of its member outcomes. A cause or outcome that links to a member event and is not a member does not feed the rating. The read lists these elements in `missingNeighbourIds`.

An event with member causes and no member outcome has no path. The Risk has no rating until an outcome joins it.

### Include and Exclude Elements

`include` lists the elements the Risk counts, and `exclude` lists elements it never counts. An empty `include` list is a Risk with no members of its own.

```yaml
risks:
  - id: R041
    label: Protection relays
    include:
      - id: E-001
      - id: C-001
      - id: O-001
    exclude:
      - C-009
```

An element is in one list at most. A Risk that lists an id twice, or in both lists, is invalid. An id that names no cause, event or outcome gets the `unknown-element` issue, and the other ids still resolve.

### Program

`filter.program` holds statements that select elements. They run over each cause, event and outcome as `item`, and they start from an empty selection, so `model |> add()` selects every element. The included ids join the result after the program runs, and the excluded ids leave it last. An element that you include or exclude therefore wins over the program.

A Risk with no executable statement and no `include` entry has no members. A statement that does not compile gets the `invalid-filter` issue.

A program adds the causes and outcomes of the events it selects only through `follow` statements:

```yaml
risks:
  - id: R040
    label: Power events and their neighbours
    include: []
    filter:
      program:
        - model |> where("item.kind == \"event\" && item.tags |> hasAny(\"power\")") |> add()
        - result |> follow("causes", 1) |> add()
        - result |> follow("outcomes", 1) |> add()
```

An element that a program or a held Risk selects shows the **Included by something else** mark in the Elements view until you include or exclude it. The mark is a grey box with a tick, and the element name is in italics.

A folder that holds an element you included or excluded shows the contains mark, whether the folder is open or closed. The mark is a box with a solid outline and the lower-left half filled. The fill is blue for an included element and maroon for an excluded one, and the box shows both colours when the folder holds both. The tooltip reads Contains included rows, Contains excluded rows, or Contains included and excluded rows.

### Hold Other Risks

A Risk holds another Risk through its `risks` list. The `flattenSubRisks` key sets how a held Risk counts. A Risk without the key flattens.

| `flattenSubRisks` | The held Risk counts as | Its own `aggregation` | Overlap with the holder |
|---|---|---|---|
| `true`, or absent | Every element beneath it, merged into the holder | Not used by the holder | Removed, because each element counts once |
| `false` | One rated item at its final values | Applies inside the held Risk | Counted twice |

Flattening takes every element beneath the held Risk, including the elements of the Risks that it holds in turn, whatever its own `flattenSubRisks` says.

```yaml
risks:
  - id: R042
    label: Both stations
    include: []
    risks:
      - R001
      - R041
    flattenSubRisks: false
```

A `risks` entry that names no Risk gets the `unknown-risk` issue. Two Risks that hold each other get the `risk-cycle` issue, and each Risk leaves out the Risk that closes the cycle.

### Folders

A cause, event, outcome or Risk can name the folder it sits in with the `folder` key. A folder path has segments separated by `/`. Each segment is non-empty and has no leading or trailing space. Two sibling folders cannot differ only by case.

The manifest lists folders in `elementFolders` and `riskFolders`, so that a folder with nothing in it stays in the model. An element or a Risk whose `folder` is not in the list still belongs to that folder.

```yaml
elementFolders:
  - Site
  - Site/Protection
riskFolders:
  - Substations
```

## Browse and Create Risks on the Canvas

The Diagram surface draws one Risk from the whole model. A Risk opens in the **Expanded** view, which draws its member events with their causes and outcomes. The **Aggregated** view draws its real causes and outcomes around one aggregated event instead. A model that has no Risk chosen shows a prompt in place of the canvas.

The Model Tree lists the Risks in their folders. A Risk with no `folder` sits at the top level. A folder row shows the last segment of its path. A folder row sorts before the Risks beside it, and each group sorts by name without regard to case. A Risk that another Risk holds does not nest under it. It sits in its own folder. A folder row shows a folder glyph, open or closed, and a Risk row shows a warning glyph. A Risk row shows the Risk ID, then the Risk name in grey. A Risk with no name shows the ID alone. The filter box matches the ID and the name. Neither row shows a type label, and pointing at a row shows its kind after its name, as in "R1 Overpressure, Risk". On the **Table** surface the Model Tree lists the elements, and each element row keeps its type label.

Click a Risk to draw it. The toolbar shows the Risk id, and **×** clears the choice. Click a folder to highlight it. A folder draws nothing, and a double-click on it does nothing. A double-click on a Risk opens **Model Properties**, which holds the default aggregation of the model.

A Risk with no members shows the note "This Risk has no elements yet." The **Aggregated** view draws its event alone.

A Risk whose events lack a member cause or a member outcome has no paths, and so no rating. The canvas still draws its member events and the member causes and outcomes linked to them. The note then says the Risk has no paths. When the events link to causes or outcomes that the Risk leaves out, it points to **Include neighbours** in the Elements view; when they link to none, such as an event with an outcome and no cause, it says there is nothing to add. The rating and the breakdowns of the aggregated event read paths only. The **Expanded** view, which a Risk opens in, draws the member events. See [Aggregated and expanded events](#aggregated-and-expanded-events).

With no Risk chosen, the canvas asks for one. A model with no Risks says so. Both messages offer **Create risk**, and the toolbar has **New Risk**.

### Model Tree Buttons

While the **Diagram** surface is in front, the Model Tree has the same buttons as the Slices view in Tetra. The Slices view also has **Close slice**. A button stages a change in the pending changes of the diagram, and **Save all** writes it. The tree shows the staged state at once: a moved Risk appears in its new folder, and a deleted Risk leaves the tree.

The toolbar above the tree has two buttons.

| Button | Effect |
|---|---|
| **New Risk** | Adds a new Risk to the pending changes and opens it in the [Risk drawer](#risk-drawer). With one folder highlighted, the Risk goes in that folder. |
| **New folder** | Adds a folder named New Folder, inside the highlighted folder when there is one. The new name is a text field. Press Enter to keep the name, or press Escape to remove the folder. |

Highlight a row to see its buttons in the toolbar. A right-click on a row opens the same buttons as a menu. A Cmd, Ctrl or Shift click highlights several rows, and a button then acts on all of them.

| Row | Button | Effect |
|---|---|---|
| Risk | **Open** | Draws the Risk, as a click does. The **×** button in the diagram toolbar clears the choice. |
| Risk | **Edit** | Opens the Risk in the [Risk drawer](#risk-drawer). The button is disabled for a Risk that the drawer cannot open. |
| Risk | **Duplicate** | Adds a copy with the next free id and the label followed by "(copy)". The copy shows in the tree at once, and the canvas draws it when you pick it, before **Save all**. |
| Risk | **Delete** | Removes the Risk. |
| Folder | **Rename** | Opens the folder name for editing. Press Enter to keep the new name, or press Escape to leave the folder as it was. The Risks and subfolders in the folder move with it. The tree refuses an empty name, a name with a slash, and a name that repeats a sibling or differs from it only by case. It shows the reason under the name and keeps the name open. |
| Folder | **New folder** | Adds a folder named New Folder inside that folder, with its name open for editing. |
| Folder | **Delete folder** | Removes a registered folder. The button is disabled, with the reason, when the folder holds a Risk or a subfolder, and when only a Risk names the folder. |

Drag a Risk or a folder onto a folder to move it, or onto the empty area to move it to the top level. A folder moves with its subfolders and its Risks. The tree refuses some moves and gives the reason in a message under the toolbar. A folder cannot go inside itself. A folder name cannot repeat a sibling, even when the names differ only by case.

A read-only model disables every button except **Open**. Each disabled button gives the reason "Editing is turned off for this model.", and the rows do not drag. The **Table** surface has none of these buttons, because the Model Tree lists the elements there.

**Model Properties** lists no Risks and no folders. Make, change, move and delete them here in the Model Tree.

### Elements View

The Elements view sits in the activity bar, below the Model Tree. It lists the causes, events and outcomes of the whole model in element folders. An element with no `folder` sits at the top level. A folder row sorts before the elements beside it, and each group sorts by name without regard to case. Controls have no row. A cause row shows an arrow, an event row a diamond and an outcome row a ring, and a folder row shows a folder glyph, open or closed. An element row shows the element ID, then its label in grey. An element with no label shows the ID alone. The filter box matches the ID and the label. No row shows a type label. Pointing at a row shows its kind after its name, as in "C1 Pump trip, cause".

The view appears when you pick a Risk on the **Diagram** surface. It hides again when you clear the Risk or switch to the **Table**, where the Model Tree lists the elements. The web shell shows the same list in an **Elements** section below the Model Tree.

| Control | Effect |
|---|---|
| **Filter by name** | Narrows the rows to the elements whose name matches, and the folders above them. |
| State toggle on a row | Cycles the row through not set, included and excluded for the picked Risk. The toggle is off in a read-only model, and its tooltip gives the reason. |
| State toggle on a folder row | Sets the state of every element in the folder at that moment. An element added to the folder later does not join. |
| Click, **Ctrl**+click or **Shift**+click on a row | Picks one row, adds or removes a row, or picks the rows between. The toggle of a picked row sets the state of every picked row. |
| **i**, **x** or **u** on a picked row | Includes the row, excludes it, or clears its state. |
| **Add element** | Opens a menu of **Add cause**, **Add event** and **Add outcome**. Choosing one opens the create form. When one folder row is picked, the new element takes that folder in its **Folder** field. Without a picked folder the element sits at the top level. The status line under the buttons names the folder until another action replaces it. |
| **Add to Risk** with rows picked | Includes each picked element, or every element in a picked folder, in the picked Risk, as one change. |
| **Include neighbours** | Includes every missing neighbour of the picked Risk. See [Include neighbours](#include-neighbours). |
| Double-click on an element | Opens the element in the drawer. |
| **Delete** with an element picked | Adds the deletion to the pending changes of the diagram. |
| **New folder** | Adds a folder named New Folder (New Folder 2 if that name is taken) to `elementFolders`, inside the picked folder when one is picked, and opens its name for editing. Enter or leaving the field keeps the name, and Escape removes the folder. The view refuses an empty name, a name that already exists, and a name that differs from an existing folder only by case, shows the reason under the name, and keeps the name open. |
| **Delete** with a folder picked | Removes a folder from `elementFolders`. It is off for a folder that holds elements or subfolders, and for a folder that exists only because an element names it. The tooltip gives the reason. |

The actions act on the picked rows. They sit in the icon buttons of the toolbar above the filter, with the name of each in its tooltip. **Add element** shows a small arrow and opens its menu, and **Add to Risk** shows a check mark, so no two buttons share a glyph. A right-click on a row opens a menu of the same row actions, picking the row first when it is not picked. An action that one of several picked rows lacks shows disabled, with the reason "Not every picked row supports this". A read-only model disables every control that changes the model, and **Add element** then stays closed with the reason in its tooltip. A change waits in the pending changes of the diagram until **Save all**. A folder row also carries **Rename**, **New folder** and **Delete folder**, as a folder in the Model Tree does. **Rename** opens the folder name for editing, and the elements and subfolders in the folder move with it. **Discard all** in the pending changes bar clears the view's notice, its highlight and any folder name that is open. The view does not take a drag onto the canvas, because the view and the canvas are separate panels.

Drag an element or a folder onto a folder to move it, or onto the empty area to move it to the top level. A folder moves with its subfolders and its elements, and a moved element takes the folder in its **Folder** field. The tree shows the new place at once, and **Save all** writes it. The view refuses some moves and gives the reason in the status line under the toolbar. A folder cannot go inside itself. A folder name cannot repeat a sibling, even when the names differ only by case. A read-only model offers no drag.

### Include Elements in a Risk

Pick a Risk. Then use one of these actions in the Elements view:

- Select the state toggle of a row.
- Pick an element row, or several, and select **Add to Risk**.
- Pick a folder row and select **Add to Risk**. The Risk includes every element whose `folder` is that path or lies beneath it.

Each action adds one change to the pending changes and keeps the other fields of the Risk. An element that the Risk excluded moves to its included elements when you add it with **Add to Risk**. The view says when the Risk already includes the element. The canvas draws the change at once, and **Save all** writes it.

A toggle that returns a Risk to its saved form drops the pending change.

A cause, event or outcome that you create while a Risk is open joins that Risk. This holds for every way to create one: **Create cause** in the Findings view, the create items of the canvas menu, **Duplicate**, and **Add element** in the Elements view. **Create** adds the element and its place in the Risk to the pending changes, and **Discard all** drops both. The canvas draws the new element at once, with no link to an event, and the rating of the Risk counts it after **Save all**. A control or an annotation does not join the Risk.

### Include Neighbours

A neighbour is a cause or outcome that links to a member event and is not a member. **Include neighbours** adds every neighbour of the picked Risk to its included elements. The tooltip of the button shows how many neighbours the Risk has. It is off for a Risk with no neighbours.

### Show Missing Neighbours

The **View** menu has a **Missing neighbours** row. **Show** draws the neighbours of the picked Risk beside its members, faded and with a dashed border. A neighbour has no controls and cannot be edited on this canvas. The level and numbers of the Risk do not change, because only members feed the rating.

### See Which Risks Include an Element

In the diagram, the drawer of a cause, event or outcome has an **Included in Risks** section that lists the Risks that include it, or says that none does. The **Table** has a **Risks** column, off by default. Turn it on in **Edit columns**. Its cell lists the Risks that include the row, and a control row has none.

## Set the Model Defaults in the Model Properties

Choose **Model Properties** in the toolbar of a Lamina diagram. The drawer holds settings of the whole model: the Tetra model, the Tetra zones in scope, and the **Default Risk aggregation** block. It lists no Risks and no Risk folders. Use the [Model Tree buttons](#model-tree-buttons) for those.

| Control | Effect |
|---|---|
| **Default Risk aggregation** | Sets the aggregation of every Risk that sets none of its own. A Risk that sets its own aggregation overrides these defaults, one setting at a time. See [Aggregation selects](#aggregation-selects). |

A change waits in the pending changes of the diagram, and **Save all** writes it. A change back to the saved value drops the pending change.

### Risk Drawer

A Risk opens in the properties drawer, as a cause or an outcome does, and the canvas stays in view. Open it with **Edit** in the Model Tree, or make a new one with **New Risk** or **Create risk**.

**New Risk** adds a Risk to the pending changes at once, with the next free id such as `R002` and the id as its label, in the folder you chose or at the top level. The Risk includes nothing yet and has `flattenSubRisks` on. The id is fixed from then on. To remove a Risk that you do not want, use **Undo** or **Discard all** in the pending changes bar. The Risk reaches the Model Tree after **Save all**, and the canvas then draws it.

| Field | Content |
|---|---|
| **Label** | Required. |
| **Id** | Fixed once the Risk exists. |
| **Folder** | A folder path, or empty for the top level. The list offers the folders that exist. A path that does not exist makes the folder. |
| **Description** | A note about the Risk. |
| **Query** | The program of the Risk. See below. |
| **Held Risks** | A list of the Risks that this Risk holds. Choose a Risk to add it, and select **×** to remove it. The list forms the `risks` list. |
| **Held Risks count as their elements** | Shown once the Risk holds a Risk. It sets `flattenSubRisks`. |
| **Aggregation** | The aggregation of this Risk. See [Aggregation selects](#aggregation-selects). |

Each change goes to the pending changes of the diagram as you make it, and **Save all** writes it. The label, folder and description fields apply when you leave the field or press Enter. A change back to the saved value drops the pending change. When a value is not allowed, the drawer shows why and keeps the last good value: an empty label, and a folder path with an empty segment or one that differs from an existing folder only by case. A read-only model disables the fields.

The program takes one of two modes. In **Tags**, choose the tags to match from the list. The Risk takes every cause, event and outcome that carries any chosen tag, and the drawer counts the elements that match. A control or an annotation that carries a tag is never a member. In **Query**, write the program. A Risk with no tag and no statement selects nothing through its program, and the drawer says so.

A program that is not exactly a tag filter opens as a **Custom query**. The tag list is off for it, and the drawer keeps the program as written. **Tags** refuses to replace a custom query.

The drawer does not list or change the elements that you include or exclude. It states how many there are. Change them in the [Elements view](#elements-view).

### Aggregation Selects

The model default and each Risk have the same four selects. A select that shows **(not set)** leaves that rule to the next level.

| Select | Sets | Values |
|---|---|---|
| **Per-event likelihood** | `perEvent.likelihood` | `max`, `simple-damped-union`, `exposure-grouped` |
| **Per-event consequence** | `perEvent.consequence` | `per-category-worst`, `all-from-worst-average` |
| **Across events: likelihood** | `crossEvent.likelihood` | `max`, `simple-damped-union` |
| **Across events: consequence** | `crossEvent.consequence` | `per-category-worst`, `all-from-worst-average` |

A **damping factor** field appears under a likelihood select that shows `simple-damped-union`. It takes a number from 0 to 1. An empty field uses the default of 0.85.

Each rule takes the first of these that is set: the Risk, the model default, and the built-in rule. The built-in per-event likelihood is `simple-damped-union`. The other three have no built-in rule, so a Risk with none set rates its worst pair.

A Risk that holds another Risk counts that Risk as one item. It does not de-duplicate an event that the Risk reaches through a held Risk and through its own members, so the event counts twice.

A change waits in the pending changes of the diagram. **Save all** writes it to the manifest with any other edit.

The Add Risk wizard, which created a cause, an event and an outcome, no longer shows in the command palette or in the canvas and table menus. **New Risk** on the Diagram toolbar creates a Risk over events that already exist.

## Models with No Risks

A manifest without a `risks` list has no Risks. The read answers an empty list, every event is in `uncoveredEventIds`, and the Diagram asks for a Risk to be created.

## Read the Risks

`GET /cross-app/models/<model-id>/risks` returns the Risks of one Lamina model. `<model-id>` is the id of the model. The [Discovery routes](/docs/sightline/reference/api/discovery) page of the API reference lists the route.

| Member | Content |
|---|---|
| `risks` | Each Risk with its id, label, events, sub-risks, aggregation, paths and rating, and its `folder`, `memberIds` and `missingNeighbourIds`. |
| `membership` | Which Risks include which element, and which Risks hold which Risk. See [Find where an element is used](#find-where-an-element-is-used). |
| `ratingBasis` | The risk matrix and the likelihood rule that the ratings used. |
| `uncoveredEventIds` | The events that no Risk includes. |
| `issues` | The definitions that the read could not fully honour. |
| `problems` | The files of the model that could not be read or parsed. |

Each issue names the Risk and, where it applies, the id at fault. A Risk with an issue still resolves, and the other Risks are unaffected.

| Issue | Meaning |
|---|---|
| `invalid-definition` | The entry does not match the Risk fields, or has no `include`. |
| `duplicate-id` | An earlier Risk has this id, and the read drops this one. |
| `no-paths` | The Risk groups no path and holds no sub-risk. |
| `unknown-risk` | A `risks` entry names a Risk that the manifest does not define. |
| `risk-cycle` | A `risks` entry names a Risk that reaches this Risk. The read leaves that entry out. |
| `unknown-element` | An `include` or `exclude` entry names an element the model does not hold. |
| `invalid-filter` | The program of a Risk does not compile, or a statement of it fails. |
| `no-members` | A Risk has no member and holds no Risk. |

Check `problems` before quoting a count. A file that fails to parse drops its causes, events and outcomes from the answer, and the counts then fall short.

### Find Where an Element Is Used

`membership.byElement` maps a cause, event or outcome id to the ids of the Risks whose members include it, in the order of the manifest. A Risk includes the elements that it counts: its members, and for a flattened held Risk the elements beneath it. An element that is only a missing neighbour is not listed. An element that no Risk includes has no entry.

`membership.byRisk` maps a Risk id to the ids of the Risks that hold it, whether they flatten it or not. A model with no `risks` list has an empty `membership`.

## Rate a Risk

A Risk with paths, or with a sub-risk that has a rating, has a rating. The rating has a residual side and an inherent side.

| Side | Likelihood | Consequence |
|---|---|---|
| Residual | `currentLikelihood` of the causes, lowered by the outcome's `likelihoodReduction` | `residualConsequence` of the outcome |
| Inherent | `inherentLikelihood` of the causes, with no reduction | `inherentConsequence` of the outcome |

Each distinct pair of an event and an outcome in the Risk gets its own rating. The causes of the pair combine into one likelihood. The outcome supplies the worst consequence of its categories. The risk matrix of the model gives the level for that likelihood and consequence.

The rating of the Risk is the level of its worst pair. A sub-risk adds one pair, at its own final likelihood and consequence. The rating also names that pair, and it lists every pair with its likelihood, consequence and level. A Risk that names one event has the same residual level that the diagram shows for that event, when the diagram uses the same aggregation rule.

A cause or outcome that takes its rating from a template is rated. A likelihood written as a letter is read as its index in the risk matrix.

A pair with a missing likelihood, a missing consequence or no matrix cell has no level. The rating of such a pair still lists the keys it has.

### Set How a Risk Aggregates

The `aggregation` key of a Risk, and the `riskDefaults` key of the manifest, set four rules. `riskDefaults` has the same form as `aggregation` and applies to a Risk that sets none of its own. A Risk replaces each rule separately.

| Rule | Effect |
|---|---|
| `perEvent.likelihood` | How the causes of one event and outcome combine. |
| `perEvent.consequence` | When set, folds the outcomes of one event into one outcome before the Risk is rated. |
| `crossEvent.likelihood` | How the likelihoods of the events and sub-risks combine. |
| `crossEvent.consequence` | How the consequences of the events and sub-risks combine. |

A likelihood rule has a `mode`. `perEvent.likelihood` takes `max`, `simple-damped-union` or `exposure-grouped`. `crossEvent.likelihood` takes `max` or `simple-damped-union`, because it combines events and not causes. The optional `dampingFactor` applies to `simple-damped-union`.

`exposure-grouped` groups the causes of an event by the source zone of the flows that they link. Causes that link no flow share one group. The optional `withinDamping` and `acrossDamping` set the damping inside a group and across groups. A model written before schema version 7 may name `exposure-vector-grouped`; the migration rewrites it to `exposure-grouped`, as [Lamina schema version 7](/docs/sightline/upgrade/compatibility#lamina-schema-version-7) describes.

A consequence rule has a `mode` of `per-category-worst` or `all-from-worst-average`. `per-category-worst` takes the highest rating of each category. `all-from-worst-average` takes the whole row with the highest average rating, and the lower id wins a tie.

```yaml
riskDefaults:
  perEvent:
    likelihood: { mode: simple-damped-union }
risks:
  - id: R030
    include:
      - id: C-001
      - id: E-001
      - id: E-002
      - id: O-001
    aggregation:
      crossEvent:
        likelihood: { mode: max }
        consequence: { mode: per-category-worst }
```

A Risk without `crossEvent` is rated by its worst pair. With `crossEvent`, the events and sub-risks collapse into one pair. Each event counts as its worst pair, unless `perEvent.consequence` folded its outcomes into one. When only one of the two `crossEvent` rules is set, the other uses `max` for likelihood and `per-category-worst` for consequence.

The collapsed pair lists no event or outcome. A folded outcome with `per-category-worst` takes the id `synth-outcome-<event id>`, and `all-from-worst-average` keeps the id of the outcome it picked. The folded outcome lowers the residual likelihood by the smallest `likelihoodReduction` among the outcomes it folds.

A `riskDefaults` block that does not match this form is ignored, and the read adds a `problems` entry that names it.

### Choose the Aggregation Rule

The causes of a pair combine under the `perEvent.likelihood` rule of the Risk. A Risk that sets none uses the `perEvent.likelihood` rule of `riskDefaults`, and without either the causes combine as `simple-damped-union`. The rule belongs to the model or the Risk.

The **Default Risk aggregation** block of the drawer sets `riskDefaults`. The read adds a `problems` entry and uses `simple-damped-union` when `riskDefaults` does not match its form.

### Rating Basis

`ratingBasis` names what the ratings used.

| Member | Content |
|---|---|
| `riskMatrixDef` | `model` when the manifest names a risk matrix file. `default` when it names none. `invalid` when the named file cannot be used. |
| `aggregationMode` | How the causes of a pair combine. |

A model with no `riskMatrixDef` has no matrix cells. Its ratings carry the likelihood and consequence keys and no level. An invalid file gives no rating for any Risk, and `problems` names the file.

## Ad Hoc Risk Queries

`POST /cross-app/models/<model-id>/risks/query` rates one Risk that the manifest does not need to define. The request body is JSON and takes one of two forms. The route only reads. It stores nothing, and a read-only token can call it.

| Form | Body | Answer |
|---|---|---|
| By id | `{ "riskId": "R001" }` | The rated Risk of that id. |
| Built | `{ "include": [{ "id": "E-001" }, { "id": "C-001" }, { "id": "O-001" }] }` | A Risk over the elements that you list. |

A Risk that you build also takes these optional members:

| Member | Content |
|---|---|
| `exclude` | The ids of elements the Risk never counts. |
| `filter` | `{ "program": [...] }`. The program runs first, as in the manifest. |
| `risks` | The ids of Risks the manifest defines that this Risk holds. The optional `flattenSubRisks` member sets how they count. |
| `id` | The id in the answer. The default is `adhoc`. The manifest must not define a Risk with this id. |
| `label` | The label in the answer. |
| `aggregation` | The aggregation of this query. It replaces `riskDefaults` for this query and has the same members as the `aggregation` of a Risk. |

This request rates two events and the elements around them together, with the worst likelihood across events:

```json
{
  "id": "station-outage",
  "include": [{ "id": "C-001" }, { "id": "E-001" }, { "id": "E-002" }, { "id": "O-001" }],
  "aggregation": { "crossEvent": { "likelihood": { "mode": "max" } } }
}
```

The answer has the members `risk`, `ratingBasis`, `issues` and `problems`. `risk` has the same fields as a Risk in the list above. `issues` holds only the issues of the queried Risk. An `include`, `exclude` or `risks` entry that names an unknown element or Risk adds an issue and does not fail the request.

The route answers 400 for a body that is not a valid query and for an `id` that the manifest already defines. It answers 404 for an unknown model or an unknown `riskId`.

## Pick a Risk

The Diagram surface draws the Risk that you pick in the **Model Tree** in the activity bar. The **Table** surface lists the whole model, and the Model Tree lists every element while the table is in front.

| Action | Effect on the Diagram |
|---|---|
| Click a tree row | Picks that Risk. |
| Double-click a tree row | Opens Model Properties, which holds the default aggregation of the model. |

The Model Tree lists the Risks, with each Risk that no other Risk holds at the top level and the Risks it holds nested under it to any depth. A Risk that two Risks hold appears under both.

The canvas has no Risk until you pick one, and shows a note that asks you to pick a Risk in the Model Tree. The picked Risk shows as a chip in the toolbar, and the **×** on the chip clears it. **Export PNG** and **Export Draw.io** stay off while no Risk is picked.

In the **Aggregated** view, the canvas draws the real causes and outcomes on the paths of the Risk, with their controls, around one aggregated event. The event carries the rating of the Risk: the aggregated likelihood, the aggregated consequence of each category and the level of the Risk, with the badge AGGREGATE. It is read-only. A double-click on a cause, an outcome or a control opens it. A double-click on the event opens its details.

In the **Aggregated** view, a Risk that holds other Risks draws one event with the rolled-up rating of the Risk, over the real causes and outcomes on the paths of every Risk it holds. To look at a held Risk by itself, pick its row under the holding Risk in the Model Tree.

The canvas draws a Risk that you changed at once, from the pending changes, and **Save all** writes what the canvas shows. **Discard all** puts the canvas back. The rating comes from the saved model files and the saved model default aggregation. A Risk that names an element you have not saved yet therefore shows that element as unknown until **Save all**, and a changed model default aggregation applies to the ratings after **Save all**.

### Aggregated and Expanded Events

The **View** menu has a **Risk events** row. A Risk opens in **Expanded**, which draws the member events of the Risk, each with its own causes, outcomes and controls. Each event shows its own likelihood, consequence, risk level and dot on the risk matrix. The Risk's aggregation settings rate them, so an event shows the same numbers that went into the rating of the Risk. An event that only a held Risk reaches shows the numbers from that Risk's settings. The aggregated event and the rating of the Risk are not drawn. Choose **Aggregated** to see the picture described above, with the rating of the Risk. The events are the model's own, so a double-click opens one and an edit works as for any element. With **Missing neighbours** on, a neighbour joins the member events it links to, and a neighbour that links to none is left out. A member event that lies on no path is drawn too. It has no causes or outcomes until its neighbours are included, and with **Missing neighbours** on the neighbours appear faded around it.

### Save and Revert the View

A change in the **View** menu applies to the diagram at once. The Risk does not keep it until you save it. While the **View** settings differ from the saved view of the picked Risk, the Diagram surface shows two buttons in the toolbar:

- **Save view** stages the current **View** settings on the picked Risk. The change reaches the manifest when you click **Save all**. A user who cannot edit the model sees this button disabled.
- **Revert view** discards the unsaved changes and restores the saved view of the Risk. It stages nothing, so a user who cannot edit the model can use it.

The saved view holds every setting in the **View** menu except the theme. The manifest keeps it in the `viewOptions` key of the Risk:

| **View** menu row | Key | Values |
|---|---|---|
| **Layout** | `compact` | `true` or `false` |
| **Tags** | `showTags` | `true` or `false` |
| **Elements** | `showElements` | `true` or `false` |
| **Focus** | `shadeUnrelated` | `true` or `false` |
| **Pulse Glow** | `pulseGlow` | `true` or `false` |
| **Missing neighbours** | `showMissingNeighbours` | `true` or `false` |
| **Risk events** | `expandRiskEvents` | `true` for **Expanded** (the default), `false` for **Aggregated** |
| **Outcome Sort** | `outcomeSort` | `default`, `id-asc`, `id-desc`, `lob` or `optimal` |
| **Annotations** | `annotationDisplay` | `full`, `id` or `hidden` |

A key that the saved view leaves out takes its default value.

Picking a Risk puts its saved view on the **View** menu. A Risk with no saved view leaves the **View** menu as it is. For that Risk the buttons show while any setting differs from the default, and **Revert view** restores the default view. A Risk with no saved view opens in **Expanded**, so choosing **Aggregated** shows the buttons. A Risk that saved **Aggregated** opens in it. If you click **Save view** and then change the view, **Revert view** restores the view that you staged. The buttons do not show in the Table surface or while no Risk is picked.

## Risk Consumers in Lamina

The control impact reads list the Risks that a control reaches, each with its rating. See [Control gap analysis](/docs/sightline/usage/lamina/control-gap-analysis).

The **Aggregated** view shows the rating of a Risk on the aggregated event. See [Pick a Risk](#pick-a-risk).

The risk reports use the rating rule of the model. The event summary of the Word, Excel and PowerPoint reports shows an event's inherent and residual likelihood, and its residual risk level, from the same rating as the read above. The causes of an event combine under the `perEvent.likelihood` rule of the Risk, or of `riskDefaults`. A report from an earlier release took the highest likelihood among the causes, so an event with several causes can show a higher likelihood or level than before. Set the rule to the `max` mode to keep the earlier figures.

The Excel risk register has one row for each Risk when the manifest defines `risks`. A row lists only the causes, events and outcomes on the paths of its Risk, and its identifier is the id of the Risk. The sub-risks and the `aggregation` of a Risk do not change its row, and a Risk that holds only sub-risks has an empty row. A model that defines no `risks` has an empty register.

A report names the Risk it runs on in its `risk` key. A report with no `risk` runs on the whole model.
