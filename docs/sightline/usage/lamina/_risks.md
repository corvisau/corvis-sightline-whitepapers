A Risk groups cause-event-outcome paths of a bow-tie diagram. A model manifest can define Risks, and a model that defines none has none. This page covers what a Risk groups and how to define one. It also covers how a Risk selects its members, how a Risk is rated, and what the read returns.

## What a Risk groups

A path runs from a cause through an event to an outcome. A cause lists the events it can lead to, and an event lists its outcomes. Each combination of a cause, one of its events and one outcome of that event is one path.

A Risk counts its members: the causes, events and outcomes that it includes, that its program selects, or that a Risk it holds counts. It groups every path through a member event, one of its member causes and one of its member outcomes.

A path can sit in more than one Risk.

## Define Risks in the manifest

Add a `risks` list to the `*.bowtie.yaml` manifest. This example defines two Risks:

```yaml
risks:
  - id: R001
    name: Station outage
    include:
      - id: C-001
      - id: E-001
      - id: O-001
  - id: R002
    name: Power events
    include: []
    filter:
      program:
        - model |> where("item.kind == \"event\" && item.tags |> hasAny(\"power\")") |> add()
```

| Field | Content |
|---|---|
| `id` | Required. Starts with a letter, then letters, digits, `.`, `_` or `-`. Each Risk in the model has its own id. |
| `name` | The display name. |
| `description` | A note about the Risk. |
| `include` | Required. The elements the Risk counts, one `id` for each. An empty list is a Risk with no members of its own. See [What a Risk includes](#what-a-risk-includes). |
| `exclude`, `filter`, `risks`, `flattenSubRisks` | The elements the Risk never counts, its program, the Risks it holds, and how a held Risk counts. See [What a Risk includes](#what-a-risk-includes). |
| `folder` | The folder the Risk sits in. See [Folders](#folders). |
| `aggregation` | How the Risk aggregates. See [Set how a Risk aggregates](#set-how-a-risk-aggregates). |

A Risk with no `include` key, or with any other key, is invalid.

A manifest of schema version 3 or earlier listed Risks as `parts` and named data slices. Lamina reads each data slice as a Risk with a program and each Risk of parts as a Risk that includes its elements when it opens the model, and refuses an edit until the model folder is migrated. The [compatibility page](/docs/sightline/upgrade/compatibility) describes the command that rewrites the files.

A manifest that defines Risks has exactly those Risks. The read lists the events that no Risk includes.

A build of Sightline from before this release refuses to load a manifest that has a `risks` list, a `riskDefaults` key or a Risk with `aggregation`.

## What a Risk includes

A Risk takes its members from three sources: the elements it includes by id, the elements its program selects, and the elements of the Risks it holds. The [Elements view](#elements-view) includes and excludes elements. The [Risk editor](#risk-editor) sets the program and the held Risks. The Model Properties drawer lists every Risk.

A member is a cause, an event or an outcome. A Risk counts its members and nothing else. A path runs through a member event, one of its member causes and one of its member outcomes. A cause or outcome that links to a member event and is not a member does not feed the rating. The read lists these elements in `missingNeighbourIds`.

An event with member causes and no member outcome has no path. The Risk has no rating until an outcome joins it.

### Include and exclude elements

`include` lists the elements the Risk counts, and `exclude` lists elements it never counts. An empty `include` list is a Risk with no members of its own.

```yaml
risks:
  - id: R041
    name: Protection relays
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
    name: Power events and their neighbours
    include: []
    filter:
      program:
        - model |> where("item.kind == \"event\" && item.tags |> hasAny(\"power\")") |> add()
        - result |> follow("causes", 1) |> add()
        - result |> follow("outcomes", 1) |> add()
```

An element that a program or a held Risk selects shows the **Included by something else** mark in the Elements view until you include or exclude it. The mark is a grey box with a tick, and the element name is in italics.

### Hold other Risks

A Risk holds another Risk through its `risks` list. The `flattenSubRisks` key sets how a held Risk counts. A Risk without the key flattens.

| `flattenSubRisks` | The held Risk counts as | Its own `aggregation` | Overlap with the holder |
|---|---|---|---|
| `true`, or absent | Every element beneath it, merged into the holder | Not used by the holder | Removed, because each element counts once |
| `false` | One rated item at its final values | Applies inside the held Risk | Counted twice |

Flattening takes every element beneath the held Risk, including the elements of the Risks that it holds in turn, whatever its own `flattenSubRisks` says.

```yaml
risks:
  - id: R042
    name: Both stations
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

## Browse and create Risks on the canvas

The Diagram surface draws one Risk from the whole model, as its real causes and outcomes around one aggregated event. A model that has no Risk chosen shows a prompt in place of the canvas.

The Model Tree lists the Risks in their folders. A Risk with no `folder` sits at the top level. A folder row shows the last segment of its path. A folder row sorts before the Risks beside it, and each group sorts by name without regard to case. A Risk that another Risk holds does not nest under it. It sits in its own folder.

Click a Risk to draw it. The toolbar shows the Risk id, and **×** clears the choice. Click a folder to highlight it. A folder draws nothing, and a double-click on it does nothing. A double-click on a Risk opens **Model Properties**.

A Risk with no members draws its event alone, with the note "This Risk has no elements yet."

With no Risk chosen, the canvas asks for one. A model with no Risks says so. Both messages offer **Create risk**, and the toolbar has **New Risk**.

### Create a Risk

**Create risk** opens a dialog.

| Field | Content |
|---|---|
| **Name** | Required. |
| **Id** | The next free id, such as `R002`. The id follows the rules in [Define Risks in the manifest](#define-risks-in-the-manifest) and cannot repeat the id of another Risk, including a Risk that waits for **Save all**. |
| **Folder** | A folder path, or empty for the top level. The list offers the folders that exist. A path that does not exist makes the folder. |
| **Description** | A note about the Risk. |

**Create** adds a Risk that includes nothing to the pending changes of the diagram, with `flattenSubRisks` on. The Risk reaches the Model Tree after **Save all**, and the canvas then draws it. The dialog refuses an empty name, an id that does not fit the rules, an id that another Risk has, and a folder path with an empty segment or one that differs from an existing folder only by case. A read-only model disables **Create**.

### Elements view

The Elements view sits in the activity bar, below the Model Tree. It lists the causes, events and outcomes of the whole model in element folders. An element with no `folder` sits at the top level. A folder row sorts before the elements beside it, and each group sorts by name without regard to case. Controls have no row.

The view appears when you pick a Risk on the **Diagram** surface. It hides again when you clear the Risk or switch to the **Table**, where the Model Tree lists the elements. The web shell shows the same list in an **Elements** section below the Model Tree.

| Control | Effect |
|---|---|
| **Filter by name** | Narrows the rows to the elements whose name matches, and the folders above them. |
| State toggle on a row | Cycles the row through not set, included and excluded for the picked Risk. The toggle is off in a read-only model, and its tooltip gives the reason. |
| State toggle on a folder row | Sets the state of every element in the folder at that moment. An element added to the folder later does not join. |
| Click, **Ctrl**+click or **Shift**+click on a row | Picks one row, adds or removes a row, or picks the rows between. The toggle of a picked row sets the state of every picked row. |
| **i**, **x** or **u** on a picked row | Includes the row, excludes it, or clears its state. |
| **Add cause**, **Add event**, **Add outcome** | Opens the create form. When one folder row is picked, the new element takes that folder in its **Folder** field. Without a picked folder the element sits at the top level. The status line under the buttons names the folder until another action replaces it. |
| **Add to Risk** with rows picked | Includes each picked element, or every element in a picked folder, in the picked Risk, as one change. |
| **Include neighbours** | Includes every missing neighbour of the picked Risk. See [Include neighbours](#include-neighbours). |
| Double-click on an element | Opens the element in the drawer. |
| **Delete** with an element picked | Adds the deletion to the pending changes of the diagram. |
| **New folder** | Adds a folder named New Folder (New Folder 2 if that name is taken) to `elementFolders`, inside the picked folder when one is picked, and opens its name for editing. Enter or leaving the field keeps the name, and Escape removes the folder. The view refuses an empty name, a name that already exists, and a name that differs from an existing folder only by case, shows the reason under the name, and keeps the name open. |
| **Delete** with a folder picked | Removes a folder from `elementFolders`. It is off for a folder that holds elements or subfolders, and for a folder that exists only because an element names it. The tooltip gives the reason. |

The actions act on the picked rows. They sit in the icon buttons of the toolbar above the filter, with the name of each in its tooltip, and a right-click on a row opens a menu of the same actions, picking the row first when it is not picked. An action that one of several picked rows lacks shows disabled, with the reason "Not every picked row supports this". A read-only model disables every control that changes the model. A change waits in the pending changes of the diagram until **Save all**. The view does not rename a folder or move an element to another folder. Edit the **Folder** field of the element for that. It does not take a drag onto the canvas, because the view and the canvas are separate panels.

### Include elements in a Risk

Pick a Risk. Then use one of these actions in the Elements view:

- Select the state toggle of a row.
- Pick an element row, or several, and select **Add to Risk**.
- Pick a folder row and select **Add to Risk**. The Risk includes every element whose `folder` is that path or lies beneath it.

Each action adds one change to the pending changes and keeps the other fields of the Risk. An element that the Risk excluded moves to its included elements when you add it with **Add to Risk**. The view says when the Risk already includes the element. The change reaches the canvas after **Save all**.

A toggle that returns a Risk to its saved form drops the pending change.

### Include neighbours

A neighbour is a cause or outcome that links to a member event and is not a member. **Include neighbours** adds every neighbour of the picked Risk to its included elements. The tooltip of the button shows how many neighbours the Risk has. It is off for a Risk with no neighbours.

### Show missing neighbours

The **View** menu has a **Missing neighbours** row. **Show** draws the neighbours of the picked Risk beside its members, faded and with a dashed border. A neighbour has no controls and cannot be edited on this canvas. The level and numbers of the Risk do not change, because only members feed the rating.

### See which Risks include an element

In the diagram, the drawer of a cause, event or outcome has an **Included in Risks** section that lists the Risks that include it, or says that none does. The **Table** has a **Risks** column, off by default. Turn it on in **Edit columns**. Its cell lists the Risks that include the row, and a control row has none.

## Edit Risks in the model properties

Choose **Model Properties** in the toolbar of a Lamina diagram. The drawer shows the **Model default aggregation** block and the list of Risks.

| Control | Effect |
|---|---|
| **Model default aggregation** | Sets the aggregation of every Risk that sets none of its own. See [Aggregation selects](#aggregation-selects). |
| **Risk folders** | Lists the folders of the risk tree, with how many Risks sit directly in each. See [Risk folders](#risk-folders). |
| **New query Risk** | Opens the [Risk editor](#risk-editor) for a new Risk. |
| **Edit** | Opens the Risk editor for a Risk. The id cannot change once the Risk is saved. |
| **Folder** | Moves the Risk to another folder, or to the top level. The list offers the registered folders and the folders that a Risk names. |
| **Delete** | Removes the Risk. |

Each row shows what the Risk selects (such as `3 included, 1 excluded, program`), its folder, and the Risks that hold it ("Held by").

### Risk folders

The **Risk folders** block lists every folder of the risk tree: the folders in `riskFolders`, the folders that a Risk names, and the folders above both.

| Control | Effect |
|---|---|
| **New folder** and **Add folder** | Adds a folder to `riskFolders`. The block refuses a path with an empty segment, a path that is already listed, and a path that differs from a listed folder only by case. |
| **Delete** | Removes a folder from `riskFolders`. It is off for a folder that holds a Risk, for a folder above another folder, and for a folder that only exists because a Risk names it. The tooltip gives the reason. |

| **Rename** | Opens a field with the full path of the folder. **Apply** renames the folder and every folder below it, and changes the `folder` of every Risk at or below it, whatever its form. The block refuses a path that already exists, a path that differs from a sibling only by case, a path with an empty segment, and a path inside the folder itself. |

A folder with nothing in it stays in the model because it is in `riskFolders`. A change waits in the pending changes of the diagram, and **Save all** writes it with any other edit. A second rename of the same folder replaces the first, and a rename back to the saved name drops the change. To move one Risk, use its **Folder** choice in the Risks list.

### Risk editor

The Risk editor makes a Risk and edits one. Open it with **New query Risk** or **Edit** in the Risks list.

| Field | Content |
|---|---|
| **Name** | Required. |
| **Id** | The next free id for a new Risk. It is fixed once the Risk exists. |
| **Folder** | A folder path, or empty for the top level. |
| **Description** | A note about the Risk. |
| **Query** | The program of the Risk. See below. |
| **Held Risks** | One tick box for each other Risk. The ticked Risks form the `risks` list. |
| **Held Risks count as their elements** | Shown once the Risk holds a Risk. It sets `flattenSubRisks`. |
| **Aggregation** | The aggregation of this Risk. See [Aggregation selects](#aggregation-selects). |

The program takes one of two modes. In **Tags**, tick the tags to match. The Risk takes every cause, event and outcome that carries any ticked tag, and the dialog counts the elements that match. A control or an annotation that carries a tag is never a member. In **Query**, write the program. A Risk with no tag and no statement selects nothing through its program, and the dialog says so.

A program that is not exactly a tag filter opens as a **Custom query**. The tag picker is off for it, and the dialog keeps the program as written. **Tags** refuses to replace a custom query.

The editor does not list or change the elements that you include or exclude. The dialog states how many there are. Change them in the [Elements view](#elements-view).

**Save** adds the change to the pending changes of the diagram, and **Save all** writes it. The dialog refuses an empty name, an id that does not fit the rules or that another Risk has, and a folder path with an empty segment or one that differs from an existing folder only by case. A read-only model disables **Save**.

### Aggregation selects

The model default and each Risk have the same four selects. A select that shows **(not set)** leaves that rule to the next level.

| Select | Sets | Values |
|---|---|---|
| **Per-event likelihood** | `perEvent.likelihood` | `max`, `simple-damped-union`, `exposure-grouped`, `exposure-vector-grouped` |
| **Per-event consequence** | `perEvent.consequence` | `per-category-worst`, `all-from-worst-average` |
| **Across events: likelihood** | `crossEvent.likelihood` | `max`, `simple-damped-union` |
| **Across events: consequence** | `crossEvent.consequence` | `per-category-worst`, `all-from-worst-average` |

A **damping factor** field appears under a likelihood select that shows `simple-damped-union`. It takes a number from 0 to 1. An empty field uses the default of 0.85.

Each rule takes the first of these that is set: the Risk, the model default, and the built-in rule. The built-in per-event likelihood is `simple-damped-union`. The other three have no built-in rule, so a Risk with none set rates its worst pair.

A Risk that holds another Risk counts that Risk as one item. It does not de-duplicate an event that the Risk reaches through a held Risk and through its own members, so the event counts twice.

A change waits in the pending changes of the diagram. **Save all** writes it to the manifest with any other edit.

The Add Risk wizard, which created a cause, an event and an outcome, no longer shows in the command palette or in the canvas and table menus. **New Risk** on the Diagram toolbar creates a Risk over events that already exist.

## Models with no Risks

A manifest without a `risks` list has no Risks. The read answers an empty list, every event is in `uncoveredEventIds`, and the Diagram asks for a Risk to be created.

## Read the Risks

`GET /cross-app/models/<model-id>/risks` returns the Risks of one Lamina model. `<model-id>` is the id of the model. The [Discovery routes](/docs/sightline/reference/api/discovery) page of the API reference lists the route.

| Member | Content |
|---|---|
| `risks` | Each Risk with its id, name, events, sub-risks, aggregation, paths and rating, and its `folder`, `memberIds` and `missingNeighbourIds`. |
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

### Find where an element is used

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

### Set how a Risk aggregates

The `aggregation` key of a Risk, and the `riskDefaults` key of the manifest, set four rules. `riskDefaults` has the same form as `aggregation` and applies to a Risk that sets none of its own. A Risk replaces each rule separately.

| Rule | Effect |
|---|---|
| `perEvent.likelihood` | How the causes of one event and outcome combine. |
| `perEvent.consequence` | When set, folds the outcomes of one event into one outcome before the Risk is rated. |
| `crossEvent.likelihood` | How the likelihoods of the events and sub-risks combine. |
| `crossEvent.consequence` | How the consequences of the events and sub-risks combine. |

A likelihood rule has a `mode`. `perEvent.likelihood` takes `max`, `simple-damped-union`, `exposure-grouped` or `exposure-vector-grouped`. `crossEvent.likelihood` takes `max` or `simple-damped-union`, because it combines events and not causes. The optional `dampingFactor` applies to `simple-damped-union`.

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

### Choose the aggregation rule

The causes of a pair combine under the `perEvent.likelihood` rule of the Risk. A Risk that sets none uses the `perEvent.likelihood` rule of `riskDefaults`, and without either the causes combine as `simple-damped-union`. The rule belongs to the model or the Risk.

The **Model default aggregation** block of the drawer sets `riskDefaults`. The read adds a `problems` entry and uses `simple-damped-union` when `riskDefaults` does not match its form.

### Rating basis

`ratingBasis` names what the ratings used.

| Member | Content |
|---|---|
| `riskMatrixDef` | `model` when the manifest names a risk matrix file. `default` when it names none. `invalid` when the named file cannot be used. |
| `aggregationMode` | How the causes of a pair combine. |

A model with no `riskMatrixDef` has no matrix cells. Its ratings carry the likelihood and consequence keys and no level. An invalid file gives no rating for any Risk, and `problems` names the file.

## Query a Risk without defining it

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
| `name` | The name in the answer. |
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
| Double-click a tree row | Opens Model Properties, where the Risks are edited. |

The Model Tree lists the Risks, with each Risk that no other Risk holds at the top level and the Risks it holds nested under it to any depth. A Risk that two Risks hold appears under both.

The canvas has no Risk until you pick one, and shows a note that asks you to pick a Risk in the Model Tree. The picked Risk shows as a chip in the toolbar, and the **×** on the chip clears it. **Export PNG** and **Export Draw.io** stay off while no Risk is picked.

The canvas draws the real causes and outcomes on the paths of the Risk, with their controls, around one aggregated event. The event carries the rating of the Risk: the aggregated likelihood, the aggregated consequence of each category and the level of the Risk, with the badge AGGREGATE. It is read-only. A double-click on a cause, an outcome or a control opens it. A double-click on the event opens its details.

A Risk that holds other Risks draws one event with the rolled-up rating of the Risk, over the real causes and outcomes on the paths of every Risk it holds. To look at a held Risk by itself, pick its row under the holding Risk in the Model Tree.

The canvas follows the saved manifest, so a Risk that you changed shows after **Save all**.

### Aggregated and expanded events

The **View** menu has a **Risk events** row. **Aggregated** is the picture above. **Expanded** draws the member events of the Risk instead, each with its own causes, outcomes and controls, and the numbers the model gives that event. The aggregated event and the rating of the Risk are not drawn. The events are the model's own, so a double-click opens one and an edit works as for any element. With **Missing neighbours** on, a neighbour joins the member events it links to, and a neighbour that links to none is left out.

## Where Risks are used

The control impact reads list the Risks that a control reaches, each with its rating. See [Control gap analysis](/docs/sightline/usage/lamina/control-gap-analysis).

The canvas shows the rating of a Risk on the aggregated event. See [Pick a Risk](#pick-a-risk).

The risk reports use the rating rule of the model. The event summary of the Word, Excel and PowerPoint reports shows an event's inherent and residual likelihood, and its residual risk level, from the same rating as the read above. The causes of an event combine under the `perEvent.likelihood` rule of the Risk, or of `riskDefaults`. A report from an earlier release took the highest likelihood among the causes, so an event with several causes can show a higher likelihood or level than before. Set the rule to the `max` mode to keep the earlier figures.

The Excel risk register has one row for each Risk when the manifest defines `risks`. A row lists only the causes, events and outcomes on the paths of its Risk, and its identifier is the id of the Risk. The sub-risks and the `aggregation` of a Risk do not change its row, and a Risk that holds only sub-risks has an empty row. A model that defines no `risks` has an empty register.

A report names the Risk it runs on in its `risk` key. A report with no `risk` runs on the whole model.
