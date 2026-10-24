A Risk groups cause-event-outcome paths of a bow-tie diagram. A model manifest can define Risks, and a model that defines none has one Risk for each event. This page covers what a Risk groups, how to define one, and what the read returns.

## What a Risk groups

A path runs from a cause through an event to an outcome. A cause lists the events it can lead to, and an event lists its outcomes. Each combination of a cause, one of its events and one outcome of that event is one path.

A Risk selects events. It takes every cause that lists those events and every outcome that those events list. An author can narrow it to named causes, named outcomes, or both.

| The Risk names | The Risk groups |
|---|---|
| Events | Every path through those events |
| Events and causes | The paths from those causes through those events |
| Events and outcomes | The paths to those outcomes through those events |
| Events, causes and outcomes | The paths from those causes, through those events, to those outcomes |

A path can sit in more than one Risk. A Risk that names one event, one cause and one outcome groups a single path.

A data slice is separate from a Risk. A data slice chooses what a diagram draws. See [Lamina Data Slices](/docs/sightline/usage/lamina/data-slices).

## Define Risks in the manifest

Add a `risks` list to the `*.bowtie.yaml` manifest. This example defines two Risks:

```yaml
risks:
  - id: R001
    name: Station outage
    events: [E-001, E-002]
  - id: R002
    name: Remote cause, one outcome
    events: [E-001]
    causes: [C-002]
    outcomes: [O-002]
```

| Field | Content |
|---|---|
| `id` | Required. Starts with a letter, then letters, digits, `.`, `_` or `-`. Each Risk in the model has its own id. |
| `name` | The display name. |
| `description` | A note about the Risk. |
| `events` | Required. The ids of the events the Risk groups. |
| `causes` | Keeps only these causes. Without it, the Risk keeps every cause. |
| `outcomes` | Keeps only these outcomes. Without it, the Risk keeps every outcome. |

A manifest that defines Risks has exactly those Risks. The read lists the events that no Risk includes.

A build of Sightline from before this release refuses to load a manifest that has a `risks` list.

## Models with no Risks

A manifest without a `risks` list has one Risk for each event. The Risk takes the id and the label of its event, and it groups every path through that event.

## Read the Risks

`GET /cross-app/models/<model-id>/risks` returns the Risks of one Lamina model. `<model-id>` is the id of the model. The [Discovery routes](/docs/sightline/reference/api/discovery) page of the API reference lists the route.

| Member | Content |
|---|---|
| `mode` | `authored` when the manifest defines Risks, and `derived` when it does not. |
| `risks` | Each Risk with its id, name, source, events and paths. |
| `uncoveredEventIds` | The events that no authored Risk includes. |
| `issues` | The definitions that the read could not fully honour. |
| `problems` | The files of the model that could not be read or parsed. |

Each issue names the Risk and, where it applies, the id at fault. A Risk with an issue still resolves, and the other Risks are unaffected.

| Issue | Meaning |
|---|---|
| `invalid-definition` | The entry does not match the Risk fields. |
| `duplicate-id` | An earlier Risk has this id, and the read drops this one. |
| `unknown-event`, `unknown-cause`, `unknown-outcome` | The model holds no event, cause or outcome with the named id. |
| `cause-not-linked`, `outcome-not-linked` | The cause or outcome exists, and no event of the Risk lists it. |
| `no-paths` | The Risk groups no path. |

Check `problems` before quoting a count. A file that fails to parse drops its causes, events and outcomes from the answer, and the counts then fall short.

A Risk carries no rating. The report, the diagram badges and the data slices do not use Risks.
