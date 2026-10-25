A Risk groups cause-event-outcome paths of a bow-tie diagram. A model manifest can define Risks, and a model that defines none has one Risk for each event. This page covers what a Risk groups, how to define one, how a Risk is rated, and what the read returns.

## What a Risk groups

A path runs from a cause through an event to an outcome. A cause lists the events it can lead to, and an event lists its outcomes. Each combination of a cause, one of its events and one outcome of that event is one path.

A Risk selects events by id, by tag or by a data slice. It groups every path through those events: every cause that lists them, and every outcome that they list.

A path can sit in more than one Risk.

A data slice is separate from a Risk. A data slice chooses what a diagram draws, and a Risk can name a data slice to select its events. See [Lamina Data Slices](/docs/sightline/usage/lamina/data-slices).

## Define Risks in the manifest

Add a `risks` list to the `*.bowtie.yaml` manifest. This example defines two Risks:

```yaml
risks:
  - id: R001
    name: Station outage
    events: [E-001, E-002]
  - id: R002
    name: Power events
    eventTags: [power]
```

| Field | Content |
|---|---|
| `id` | Required. Starts with a letter, then letters, digits, `.`, `_` or `-`. Each Risk in the model has its own id. |
| `name` | The display name. |
| `description` | A note about the Risk. |
| `events` | The ids of the events the Risk groups. |
| `eventTags` | The tags that an event must all carry to join the Risk. |
| `dataSlice` | The id of a data slice whose filter selects events. |

A Risk names at least one of `events`, `eventTags` and `dataSlice`. A Risk with any other key is invalid.

A manifest that defines Risks has exactly those Risks. The read lists the events that no Risk includes.

A build of Sightline from before this release refuses to load a manifest that has a `risks` list.

## Select events by tag or data slice

A Risk groups the events that any of its three selectors picks, and it groups each event once. The selectors combine as a union, so a Risk can list an event and also name a data slice.

| Selector | Picks |
|---|---|
| `events` | The events with these ids. |
| `eventTags` | The events that carry every listed tag. For any one of several tags, use a data slice. |
| `dataSlice` | The events that the filter of the named data slice selects. |

This example selects events in three ways:

```yaml
dataSlices:
  - id: remote-access
    filter: event.tags |> hasTags("remote")
risks:
  - id: R010
    name: Power events
    eventTags: [power]
  - id: R011
    name: Remote access, plus one event
    events: [E-009]
    dataSlice: remote-access
```

A `dataSlice` names the `id` of an entry in the `dataSlices` list of the manifest. The Risk applies the filter of that slice to the events. A slice with no filter selects every event, as it draws every event in a diagram. The Risk does not follow the data slice that a diagram has active.

The filter has the form of the `filter` of a data slice: an expression, a list of expressions, or a `program` list. An event joins the Risk when it passes any expression in a list. A `program` filter selects the events in the result of the program. A line that ends in `|> exclude()` removes the events it matches, even when another line kept them. The `cascade()` marker has no effect, because a Risk already takes the causes and outcomes of its events.

The filter selects events only. An expression that names causes, outcomes or controls instead of `event` does not apply, and the read reports `slice-not-event`.

A selection follows the model. An event that gains a tag, or starts to match the filter of a data slice, joins the Risk on the next read, and every count changes with it.

A build of Sightline from before this release refuses to load a manifest in which a Risk uses `eventTags` or `dataSlice`.

## Edit Risks in the model properties

Choose **Model Properties** in the toolbar of a Lamina diagram. The drawer opens with the **Aggregation** picker, which chooses the aggregation that the diagram shows and does not affect the Risks. Below it, the drawer lists the Risks of the model, with the **Risk aggregation** setting above the list.

| Control | Effect |
|---|---|
| **Risk aggregation** | Chooses the aggregation profile that rates every Risk. The default option removes `riskAggregationId` from the manifest. |
| **New Risk** | Opens the form with the next free id. |
| **Edit** | Opens the form for that Risk. The id cannot change once the Risk is saved. |
| **Delete** | Removes the Risk. |

The form has one field for each field of a Risk. **Events** and **Event tags** take one entry at a time: type it and press Enter, or pick a suggestion. **Data slice** lists the slices of the model.

**Done** checks the form. A Risk needs a new id, and at least one event, tag or data slice. The form flags an id that the model does not hold. The author can still save it, because a Risk can name an entity that the same commit adds.

A change waits in the pending changes of the diagram. **Save all** writes it to the manifest with any other edit.

The **Add Risk…** command of the canvas and table menus is a separate feature. It starts a wizard that creates a cause, an event and an outcome. **New Risk** defines a Risk over events that already exist.

## Models with no Risks

A manifest without a `risks` list has one Risk for each event. The Risk takes the id and the label of its event, and it groups every path through that event.

## Read the Risks

`GET /cross-app/models/<model-id>/risks` returns the Risks of one Lamina model. `<model-id>` is the id of the model. The [Discovery routes](/docs/sightline/reference/api/discovery) page of the API reference lists the route.

| Member | Content |
|---|---|
| `mode` | `authored` when the manifest defines Risks, and `derived` when it does not. |
| `risks` | Each Risk with its id, name, source, events, paths and rating. |
| `ratingBasis` | The risk matrix and the likelihood rule that the ratings used. |
| `uncoveredEventIds` | The events that no authored Risk includes. |
| `issues` | The definitions that the read could not fully honour. |
| `problems` | The files of the model that could not be read or parsed. |

Each issue names the Risk and, where it applies, the id at fault. A Risk with an issue still resolves, and the other Risks are unaffected.

| Issue | Meaning |
|---|---|
| `invalid-definition` | The entry does not match the Risk fields, or names no way to select events. |
| `duplicate-id` | An earlier Risk has this id, and the read drops this one. |
| `unknown-event` | The model holds no event with the named id. |
| `no-paths` | The Risk groups no path. |
| `unknown-slice` | The manifest holds no data slice with the id that `dataSlice` names. |
| `slice-not-event` | An expression of a data slice filter is about causes, outcomes or controls, and the read does not apply it. |
| `invalid-slice-filter` | A data slice filter does not compile, or every statement of its program fails. The Risk selects nothing from it. |
| `no-events` | A tag or data slice of the Risk matches no event. The Risk can still group events from its other selectors. |

Check `problems` before quoting a count. A file that fails to parse drops its causes, events and outcomes from the answer, and the counts then fall short.

## Rate a Risk

A Risk with paths has a rating. The rating has a residual side and an inherent side.

| Side | Likelihood | Consequence |
|---|---|---|
| Residual | `currentLikelihood` of the causes, lowered by the outcome's `likelihoodReduction` | `residualConsequence` of the outcome |
| Inherent | `inherentLikelihood` of the causes, with no reduction | `inherentConsequence` of the outcome |

Each distinct pair of an event and an outcome in the Risk gets its own rating. The causes of the pair combine into one likelihood. The outcome supplies the worst consequence of its categories. The risk matrix of the model gives the level for that likelihood and consequence.

The rating of the Risk is the level of its worst pair. The rating also names that pair, and it lists every pair with its likelihood, consequence and level. A Risk that names one event has the same residual level that the diagram shows for that event, when the diagram uses the same aggregation rule.

A cause or outcome that takes its rating from a template is rated. A likelihood written as a letter is read as its index in the risk matrix.

A pair with a missing likelihood, a missing consequence or no matrix cell has no level. The rating of such a pair still lists the keys it has.

### Choose the aggregation rule

The causes of a pair combine under one aggregation profile. The `riskAggregationId` key of the manifest names the profile, by the `id` of an entry in `aggregations`. The rule belongs to the model, so the active data slice does not change it. The **Risk aggregation** setting of the Model Properties drawer writes this key.

```yaml
riskMatrixDef: risk-matrix.def.yaml
riskAggregationId: worst-cause
aggregations:
  - id: worst-cause
    name: Worst cause
    config:
      likelihood: { mode: max, collapse: false }
```

Only the `likelihood` block of the profile applies. Without `riskAggregationId`, the causes combine as `simple-damped-union`. The read adds a `problems` entry and uses `simple-damped-union` when the id names no profile, or when the profile has no valid `likelihood` block.

A build of Sightline from before this release refuses to load a manifest that has a `riskAggregationId` key.

### Rating basis

`ratingBasis` names what the ratings used.

| Member | Content |
|---|---|
| `riskMatrixDef` | `model` when the manifest names a risk matrix file. `default` when it names none. `invalid` when the named file cannot be used. |
| `aggregationId` | The profile that `riskAggregationId` selected. The member is absent when the default applies. |
| `aggregationMode` | How the causes of a pair combine. |

A model with no `riskMatrixDef` has no matrix cells. Its ratings carry the likelihood and consequence keys and no level. An invalid file gives no rating for any Risk, and `problems` names the file.

## Where Risks are used

The control impact reads list the Risks that a control reaches, each with its rating. See [Control gap analysis](/docs/sightline/usage/lamina/control-gap-analysis).

The canvas shows the same rating. In a view whose aggregation profile combines likelihood, an event shows its residual risk level, the risk matrix plots one dot for the event, and an outcome shows its level. Each is the rating of the Risk that groups that one event or that one outcome. The causes combine under the profile that `riskAggregationId` names, not under the profile that the view has active. The likelihood text on an event node still follows the active profile, so the level and the text can disagree when the two profiles differ. A view that does not combine likelihood shows no level.

The risk reports use the rating rule of the model. The event summary of the Word, Excel and PowerPoint reports shows an event's inherent and residual likelihood, and its residual risk level, from the same rating as the read above. The causes of an event combine under the aggregation profile that `riskAggregationId` names. A report from an earlier release took the highest likelihood among the causes, so an event with several causes can show a higher likelihood or level than before. Set the profile to one with the `max` mode to keep the earlier figures.

The Excel risk register has one row for each Risk when the manifest defines `risks`. A row lists only the causes, events and outcomes on the paths of its Risk, and its identifier is the id of the Risk. A model that defines no `risks` keeps one row for each data slice.

A report resolves its Risks over the events that the data slice of the report draws. A Risk can therefore group fewer events in a report than the read returns.
