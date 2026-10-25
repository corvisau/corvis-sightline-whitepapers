A Risk groups cause-event-outcome paths of a bow-tie diagram. A model manifest can define Risks, and a model that defines none has one Risk for each event. This page covers what a Risk groups and how to define one. It also covers how a Risk is built from parts, how a Risk is rated, and what the read returns.

## What a Risk groups

A path runs from a cause through an event to an outcome. A cause lists the events it can lead to, and an event lists its outcomes. Each combination of a cause, one of its events and one outcome of that event is one path.

A Risk lists parts: data slices, events and other Risks. It groups every path through the events of its data slice and event parts: every cause that lists them, and every outcome that they list. A part that names a Risk holds that Risk whole.

A path can sit in more than one Risk.

A data slice is separate from a Risk. A data slice chooses what a diagram draws, and a Risk can name a data slice to select its events. See [Lamina Data Slices](/docs/sightline/usage/lamina/data-slices).

## Define Risks in the manifest

Add a `risks` list to the `*.bowtie.yaml` manifest. This example defines two Risks:

```yaml
risks:
  - id: R001
    name: Station outage
    parts:
      - event: E-001
      - event: E-002
  - id: R002
    name: Power events
    parts:
      - dataSlice: power
```

| Field | Content |
|---|---|
| `id` | Required. Starts with a letter, then letters, digits, `.`, `_` or `-`. Each Risk in the model has its own id. |
| `name` | The display name. |
| `description` | A note about the Risk. |
| `parts` | Required. A list of data slices, events and other Risks. See [Build a Risk from parts](#build-a-risk-from-parts). |
| `aggregation` | How the Risk aggregates. See [Set how a Risk aggregates](#set-how-a-risk-aggregates). |

A Risk lists at least one part. A Risk with any other key is invalid.

A manifest of schema version 2 selected events with the keys `events`, `eventTags` and `dataSlice`. Lamina reads each of them as parts when it opens the model, and the [compatibility page](/docs/sightline/upgrade/compatibility) describes the command that rewrites the file.

A manifest that defines Risks has exactly those Risks. The read lists the events that no Risk includes.

A build of Sightline from before this release refuses to load a manifest that has a `risks` list.

## Select events by tag or data slice

A data slice part picks the events that the filter of the named slice selects. An event part picks one event by id. A Risk groups the events that any of its parts picks, and it groups each event once, so a Risk can list an event and also name a data slice.

To select events by tag, write a data slice whose filter tests the tags and name it in a part. `hasAll` requires every listed tag. `hasAny` requires at least one.

```yaml
dataSlices:
  - id: remote-access
    filter: event.tags |> hasAny("remote")
  - id: power-and-critical
    filter: event.tags |> hasAll("power", "critical")
risks:
  - id: R010
    name: Power and critical events
    parts:
      - dataSlice: power-and-critical
  - id: R011
    name: Remote access, plus one event
    parts:
      - dataSlice: remote-access
      - event: E-009
```

A `dataSlice` part names the `id` of an entry in the `dataSlices` list of the manifest. The Risk applies the filter of that slice to the events. A slice with no filter selects every event, as it draws every event in a diagram. The Risk does not follow the data slice that a diagram has active.

The filter has the form of the `filter` of a data slice: an expression, a list of expressions, or a `program` list. An event joins the Risk when it passes any expression in a list. A `program` filter selects the events in the result of the program. A line that ends in `|> exclude()` removes the events it matches, even when another line kept them. The `cascade()` marker has no effect, because a Risk already takes the causes and outcomes of its events.

The filter selects events only. An expression that names causes, outcomes or controls instead of `event` does not apply, and the read reports `slice-not-event`.

A selection follows the model. An event that gains a tag, or starts to match the filter of a data slice, joins the Risk on the next read, and every count changes with it.

## Build a Risk from parts

The `parts` list builds a Risk from data slices, events and other Risks. Each entry has exactly one key.

| Part | Contributes |
|---|---|
| `dataSlice` | The events that the filter of the named data slice selects. |
| `event` | The event with this id. |
| `risk` | The Risk with this id, as one item. |

```yaml
risks:
  - id: R020
    name: Remote access
    parts:
      - dataSlice: remote-access
      - event: E-009
  - id: R021
    name: Station outage
    parts:
      - event: E-001
  - id: R022
    name: Site risk
    parts:
      - event: E-020
      - risk: R020
      - risk: R021
```

The slice and event parts combine as a union, and the Risk groups each event once. A cause or outcome reaches the Risk through whichever event brought it in.

A part that names a Risk holds that Risk as a sub-risk. The sub-risk stays whole. Its own events are not added to the paths of the Risk that holds it, and its own aggregation settings apply inside it. The Risk that holds it rates the final values of the sub-risk as one item.

An event that the Risk reaches through a sub-risk and through another part counts twice. The read does not remove the overlap, so the author chooses the parts to avoid it.

A Risk that holds only sub-risks has no paths of its own. It has a rating when a sub-risk has one.

A part that names a Risk the manifest does not define gets the `unknown-risk` issue. Two Risks that hold each other get the `risk-cycle` issue. Each Risk on the cycle leaves out the part that closes it, and both Risks still resolve.

A build of Sightline from before this release refuses to load a manifest with a `riskDefaults` key. It also refuses a manifest in which a Risk uses `parts` or `aggregation`.

## Edit Risks in the model properties

Choose **Model Properties** in the toolbar of a Lamina diagram. The drawer shows the **Model default aggregation** block and the list of Risks.

| Control | Effect |
|---|---|
| **Model default aggregation** | Sets the aggregation of every Risk that sets none of its own. See [Aggregation selects](#aggregation-selects). |
| **New Risk** | Opens the form with the next free id. |
| **Edit** | Opens the form for that Risk. The id cannot change once the Risk is saved. |
| **Delete** | Removes the Risk. |

The form has these fields:

| Field | Content |
|---|---|
| **Id**, **Name**, **Description** | The identity of the Risk. |
| **Parts** | The data slices, events and Risks that the Risk holds. **Add part** adds a row. Each row has a kind (**Event**, **Data slice** or **Risk**) and one id from the model. **Remove** deletes the row. |
| **Aggregation** | The aggregation of this Risk. See [Aggregation selects](#aggregation-selects). |

The list of a **Risk** part omits the Risk that the form edits, so a Risk cannot hold itself. **Done** refuses a part that makes two or more Risks hold each other, and names them.

**Done** checks the form. A Risk needs a new id and at least one part. The form flags an id that the model does not hold. The author can still save it, because a Risk can name an entity that the same commit adds.

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

A Risk that holds another Risk counts that Risk as one item. It does not de-duplicate an event that the Risk reaches through a held Risk and through its own parts, so the event counts twice.

A change waits in the pending changes of the diagram. **Save all** writes it to the manifest with any other edit.

The **Add Risk…** command of the canvas and table menus is a separate feature. It starts a wizard that creates a cause, an event and an outcome. **New Risk** defines a Risk over events that already exist.

## Models with no Risks

A manifest without a `risks` list has one Risk for each event. The Risk takes the id and the label of its event, and it groups every path through that event.

## Read the Risks

`GET /cross-app/models/<model-id>/risks` returns the Risks of one Lamina model. `<model-id>` is the id of the model. The [Discovery routes](/docs/sightline/reference/api/discovery) page of the API reference lists the route.

| Member | Content |
|---|---|
| `mode` | `authored` when the manifest defines Risks, and `derived` when it does not. |
| `risks` | Each Risk with its id, name, source, events, sub-risks, aggregation, paths and rating. |
| `ratingBasis` | The risk matrix and the likelihood rule that the ratings used. |
| `uncoveredEventIds` | The events that no authored Risk includes. |
| `issues` | The definitions that the read could not fully honour. |
| `problems` | The files of the model that could not be read or parsed. |

Each issue names the Risk and, where it applies, the id at fault. A Risk with an issue still resolves, and the other Risks are unaffected.

| Issue | Meaning |
|---|---|
| `invalid-definition` | The entry does not match the Risk fields, or lists no part. |
| `duplicate-id` | An earlier Risk has this id, and the read drops this one. |
| `unknown-event` | The model holds no event with the named id. |
| `no-paths` | The Risk groups no path and holds no sub-risk. |
| `unknown-slice` | The manifest holds no data slice with the id that a `dataSlice` part names. |
| `slice-not-event` | An expression of a data slice filter is about causes, outcomes or controls, and the read does not apply it. |
| `invalid-slice-filter` | A data slice filter does not compile, or every statement of its program fails. The Risk selects nothing from it. |
| `no-events` | A data slice part of the Risk matches no event. The Risk can still group events from its other parts. |
| `unknown-risk` | A `risk` part names a Risk that the manifest does not define. |
| `risk-cycle` | A `risk` part names a Risk that reaches this Risk. The read leaves that part out. |

Check `problems` before quoting a count. A file that fails to parse drops its causes, events and outcomes from the answer, and the counts then fall short.

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
    parts:
      - event: E-001
      - event: E-002
    aggregation:
      crossEvent:
        likelihood: { mode: max }
        consequence: { mode: per-category-worst }
```

A Risk without `crossEvent` is rated by its worst pair. With `crossEvent`, the events and sub-risks collapse into one pair. Each event counts as its worst pair, unless `perEvent.consequence` folded its outcomes into one. When only one of the two `crossEvent` rules is set, the other uses `max` for likelihood and `per-category-worst` for consequence.

The collapsed pair lists no event or outcome. A folded outcome with `per-category-worst` takes the id `synth-outcome-<event id>`, and `all-from-worst-average` keeps the id of the outcome it picked. The folded outcome lowers the residual likelihood by the smallest `likelihoodReduction` among the outcomes it folds.

A `riskDefaults` block that does not match this form is ignored, and the read adds a `problems` entry that names it.

### Choose the aggregation rule

The causes of a pair combine under the `perEvent.likelihood` rule of the Risk. A Risk that sets none uses the `perEvent.likelihood` rule of `riskDefaults`, and without either the causes combine as `simple-damped-union`. The rule belongs to the model or the Risk, so the active data slice does not change it.

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
| By id | `{ "riskId": "R001" }` | The rated Risk of that id. A model with no `risks` list has a Risk for each event, so `riskId` can be an event id. |
| From parts | `{ "parts": [{ "event": "E-001" }, { "risk": "R001" }] }` | A Risk built from the parts, rated as a Risk of the manifest would be. |

A Risk built from parts also takes these optional members:

| Member | Content |
|---|---|
| `id` | The id in the answer. The default is `adhoc`. The manifest must not define a Risk with this id. |
| `name` | The name in the answer. |
| `aggregation` | The aggregation of this query. It replaces `riskDefaults` for this query and has the same members as the `aggregation` of a Risk. |

This request rates the events `E-001` and `E-002` together, with the worst likelihood across events:

```json
{
  "id": "station-outage",
  "parts": [{ "event": "E-001" }, { "event": "E-002" }],
  "aggregation": { "crossEvent": { "likelihood": { "mode": "max" } } }
}
```

The answer has the members `risk`, `ratingBasis`, `issues` and `problems`. `risk` has the same fields as a Risk in the list above. `issues` holds only the issues of the queried Risk. A part that names an unknown event, data slice or Risk adds an issue and does not fail the request.

The route answers 400 for a body that is not a valid query and for an `id` that the manifest already defines. It answers 404 for an unknown model or an unknown `riskId`.

## Pick a Risk and swap the canvas

The toolbar has a **Model** | **Risk** control beside **Diagram** | **Table**. It shows in the Diagram surface when the model has at least one Risk. It sets the view of the canvas, and the **Model Tree** in the activity bar follows it.

| | **Model** view | **Risk** view |
|---|---|---|
| Canvas | The active data slice, as before. A picked Risk does not filter it. | The picked Risk, drawn from the whole model. |
| Data slice | The one filter. The **Data slice** menu shows. | Does not apply. The menu is hidden, and the canvas draws even in a model with no data slice. |
| Model Tree | The causes, events, outcomes and controls of the active data slice. A row that the slice leaves out is not listed. | The Risks, each Risk that no other Risk holds at the top level, with the Risks it holds nested under it to any depth. A Risk that two Risks hold appears under both. |
| Click a tree row | Selects the element and centres the canvas on it. | Picks that Risk. |
| Double-click a tree row | Opens the element. | Opens Model Properties, where the Risks are edited. |

The **Table** surface ignores the view. It lists the whole model, and the Model Tree lists every element while the table is in front.

In the **Risk** view the canvas has no Risk until you pick one, and shows a note that asks you to pick a Risk in the Model Tree. The picked Risk shows as a chip in the toolbar, and the **×** on the chip clears it. The chip shows in the **Risk** view only. The picked Risk stays when you switch to the **Model** view and back.

The **Risk** view draws the real causes and outcomes on the paths of the Risk, with their controls, around one aggregated event. The event carries the rating of the Risk: the aggregated likelihood, the aggregated consequence of each category and the level of the Risk, with the badge AGGREGATE. It is read-only. A double-click on a cause, an outcome or a control opens it as in the **Model** view. A double-click on the event opens its details.

A Risk that holds other Risks draws one event with the rolled-up rating of the Risk, over the real causes and outcomes on the paths of every Risk it holds. To look at a held Risk by itself, pick its row under the holding Risk in the Model Tree.

The **Risk** view follows the saved manifest, so a Risk that you changed shows after **Save all**.

## Where Risks are used

The control impact reads list the Risks that a control reaches, each with its rating. See [Control gap analysis](/docs/sightline/usage/lamina/control-gap-analysis).

The canvas shows the rating of a Risk on the aggregated event of the **Risk** view. See [Pick a Risk and swap the canvas](#pick-a-risk-and-swap-the-canvas). The **Model** view draws the real events and outcomes of the active data slice and shows no residual risk level for them.

The risk reports use the rating rule of the model. The event summary of the Word, Excel and PowerPoint reports shows an event's inherent and residual likelihood, and its residual risk level, from the same rating as the read above. The causes of an event combine under the `perEvent.likelihood` rule of the Risk, or of `riskDefaults`. A report from an earlier release took the highest likelihood among the causes, so an event with several causes can show a higher likelihood or level than before. Set the rule to the `max` mode to keep the earlier figures.

The Excel risk register has one row for each Risk when the manifest defines `risks`. A row lists only the causes, events and outcomes on the paths of its Risk, and its identifier is the id of the Risk. The sub-risks and the `aggregation` of a Risk do not change its row, and a Risk that holds only sub-risks has an empty row. A model that defines no `risks` keeps one row for each data slice.

A report resolves its Risks over the events that the data slice of the report draws. A Risk can therefore group fewer events in a report than the read returns.
