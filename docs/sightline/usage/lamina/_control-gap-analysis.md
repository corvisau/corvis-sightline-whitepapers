A Lamina model holds each control under the cause or outcome that owns it. The control impact reads of the Sightline API list the Risks that one control is a gap for and those it is critical for. A gap analysis paper can then quote one count and one list, and the author does not trace each cause by hand.

The routes are listed in the [Discovery routes](/docs/sightline/reference/api/discovery) page of the API reference. This page explains what they count.

## What a read counts

A control is reused through a control template. Each control that names the template is an instance of it, and the reads treat the template as the control.

A Risk groups cause-event-outcome paths. See [Risks](/docs/sightline/usage/lamina/risks). A control reaches a Risk when a path of the Risk runs through the owner of the control. The owner of a preventative control is a cause, and the owner of a mitigative control is an outcome. A Risk with no paths is reached through its events. A model that defines no Risks has one Risk for each event, with the id of the event. A Risk that several controls reach counts once.

Each instance has a state, from its target rating and its current rating:

| State | Condition | Counts as a gap |
|---|---|---|
| Gap | The current rating is below the target on None, Low, Partial and Full. | Yes |
| Unassessed | Either rating is missing or reads Not Assessed. | Yes |
| Met | The current rating reaches the target. | No |
| Not applicable | Either rating reads Not Applicable. | No |

The reads take the ratings and the critical flag from the control after its template applies. A control that omits them carries the values of the template, and a value on the control overrides the template.

A Risk is a gap risk when a control that reaches it is a gap or unassessed. A Risk is a critical risk when a control that reaches it has the critical flag. The same template can be critical on one cause and not on another. A critical risk is therefore a Risk reached through a control that carries the flag.

## Choose the control

| To read | Route |
|---|---|
| Every control that names one template | `GET /cross-app/models/<model-id>/control-templates/<template-id>/impact` |
| One control that a cause owns | `GET /cross-app/models/<model-id>/causes/<cause-id>/controls/<control-id>/impact` |
| One control that an outcome owns | `GET /cross-app/models/<model-id>/outcomes/<outcome-id>/controls/<control-id>/impact` |

`<model-id>` is the id of the Lamina model. Add `?scope=template` to either instance route to answer for every control of the same template. A control that names no template answers for itself alone.

Add `?zone=<zone-id>` to any route to keep the controls whose zones include that zone. A control takes its own zones, else the zones of its cause or outcome. Use the filter when one template covers several kinds of asset and the paper concerns one.

A model, cause, outcome, control or template that does not exist answers 404, and the error names the missing id. A control that belongs to another cause or outcome does not match. Two models with the same id answer 409. A `scope` other than `instance` or `template` answers 400.

## Read the answer

| Member | Content |
|---|---|
| `control` | The template id, with the label, type, category and group of the first control. |
| `instances` | Each matched control: its owner, ratings, state, critical flag, zones and the events it reaches. |
| `risks` | Each Risk reached: its id, name, source, events and rating, the controls that reach it, and its gap and critical flags. |
| `summary` | The number of controls and Risks, the ids of the gap and critical Risks, and the number of controls in each state. |
| `ratingBasis` | The risk matrix and the likelihood rule that the ratings used. |
| `issues` | The Risk definitions that the read could not fully honour. |
| `problems` | The files of the model that could not be read or parsed. |

The rating of a Risk has a residual side and an inherent side, as described in [Risks](/docs/sightline/usage/lamina/risks). A Risk with no paths has no rating.

Check `problems` before quoting a count. A file that fails to parse drops its causes, events and outcomes from the answer, and the counts then fall short.

This trimmed answer comes from a template that two causes use. One control is a gap and the other meets its target:

```json
{
  "control": { "templateId": "tpl-access-local", "label": "Local access control" },
  "instances": [
    { "controlId": "K-001", "ownerKind": "cause", "ownerId": "C-001", "state": "gap", "critical": true, "eventIds": ["E-001", "E-002"] },
    { "controlId": "K-002", "ownerKind": "cause", "ownerId": "C-002", "state": "met", "critical": true, "eventIds": ["E-002"] }
  ],
  "summary": {
    "instanceCount": 2,
    "riskCount": 2,
    "gapRiskIds": ["E-001", "E-002"],
    "criticalRiskIds": ["E-001", "E-002"],
    "byState": { "gap": 1, "met": 1, "unassessed": 0, "not-applicable": 0 }
  },
  "ratingBasis": { "riskMatrixDef": "model", "aggregationMode": "simple-damped-union" },
  "issues": [],
  "problems": []
}
```

The model in this example defines no Risks, so each event is a Risk with the id of the event. Both Risks are gap risks, because `K-001` reaches both and is a gap.
