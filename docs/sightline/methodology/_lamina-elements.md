A Lamina model describes risk as a bow-tie diagram. An event sits in the centre. The causes that can lead to it sit on the left, and the outcomes that can follow it sit on the right. Controls sit on the lines between them. Each part of the model has one job, and a model stays readable when each element does only that job.

## The elements

| Element | What it represents | Owns |
|---|---|---|
| Event | The moment control is lost, between its causes and its outcomes | Nothing; it lists the outcomes it can produce |
| Cause | A way the event can start | Its actor, vector, exposure, likelihoods and controls; it lists the events it can lead to |
| Outcome | A result that can follow the event | Its consequence ratings and controls |
| Control | A barrier on the line between a cause and the event, or between the event and an outcome | The fields of its kind. See [Controls](#controls) |
| Risk | A group of cause-event-outcome paths that is rated as one | Nothing; it names its members. See [Risks](/docs/sightline/usage/lamina/risks) |

A path runs from a cause through one of its events to one outcome of that event. A cause can point at several events, and several causes can point at one event. An event can point at several outcomes, and several events can point at one outcome. These links are references, so one outcome serves every event that names it.

## Causes, events and outcomes

Choose the event first. An event is the point where a hazard is released or control of a system is lost. An example is loss of control of a pumping station. An event sits between the trigger and the damage. A candidate event that reads as a trigger belongs on the left as a cause. A candidate that reads as damage belongs on the right as an outcome.

A cause is one specific way the event can start, such as malware brought in on a contractor laptop. Give each distinct route its own cause, because each cause carries its own likelihoods and its own preventative controls.

An outcome is one specific result of the event, such as an untreated discharge to a river. An outcome carries its consequence ratings, one for each consequence category of the risk matrix. It also records the lines of business it affects and whether you judge it a material risk. Neither of those two fields changes the rating.

## Actor, vector and exposure

A cause describes its threat with three attributes. Each one answers a different question.

| Attribute | Question it answers | Fields |
|---|---|---|
| Actor | Who or what can trigger the cause? | Intent, capability and motivation |
| Vector | By which route does the cause reach the target? | Complexity, source zones and interfaces |
| Exposure | What is the cause able to reach, and in which zones? | Zones |

A cause owns its actor, vector and exposure. Reuse comes from a template, so two causes with the same actor each name the same actor template. A vector can link the records that Tetra rules create for a channel or flow, and Lamina reports when a linked record changes. See [Record links](/docs/sightline/usage/lamina/finding-links).

Each attribute can carry a base likelihood and a list of suggested control templates. The base likelihood is a starting point to read when you set the likelihoods of the cause. The rating reads the likelihoods of the cause, never the base likelihood of an attribute. A suggested control template appears in the drawer of a cause that uses the attribute, ready to add.

The exposure and vector of a cause also group the causes of an event when the model uses a grouped likelihood rule. See [Choose the aggregation rule](/docs/sightline/usage/lamina/risks#choose-the-aggregation-rule).

## Controls

A control belongs to the one cause or outcome that owns it. A control on a cause stops the cause from reaching the event. A control on an outcome limits the outcome once the event has happened.

A control is one of three kinds.

| Kind | Owner | What it records | Choose it when |
|---|---|---|---|
| Assessed control | Cause or outcome | Category, type, target and current effectiveness, a critical flag, and the elements it covers | You rate how well the barrier works against a target |
| Inherent control | Cause or outcome | Whether the baseline control holds | The barrier exists before any assessed control, and you only record that it is in place and working |
| Conditional modifier | Outcome only | A free-text condition | The barrier counts only when a stated condition holds |

An assessed control has a type that says what it does about the risk.

| Type | What the control does |
|---|---|
| Preventative | Stops a cause from reaching the event |
| Mitigative | Limits an outcome after the event |
| Detective | Detects that the cause or the event is happening, without stopping or limiting it by itself |
| Not Applicable | None of the above fits |

An assessed control also has a category: Technology for a technical measure, Process for a procedure, and People for something that people do.

Mark a control as critical when the risk depends on it. The critical flag and the effectiveness ratings feed the control gap analysis. See [Control gap analysis](/docs/sightline/usage/lamina/control-gap-analysis).

A conditional modifier never changes the residual rating, and Lamina does not evaluate its condition. Effectiveness ratings do not change the rating either. They record the state of each barrier, and you carry that judgement into the likelihoods and consequences below. See [Authoring controls](/docs/sightline/usage/lamina/authoring-controls).

## Inherent and residual

A rating has two sides. The inherent side rates the risk with no controls in place. The residual side rates it with the controls in place today.

| Side | Likelihood | Consequence |
|---|---|---|
| Inherent | The inherent likelihood of each cause | The inherent consequence of each outcome |
| Residual | The current likelihood of each cause, lowered by the likelihood reduction of the outcome | The residual consequence of each outcome |

Set the inherent likelihood of a cause as if no control acted. Set its current likelihood with the preventative controls of today in place. On an outcome, set the inherent consequence before its mitigative controls act and the residual consequence after they act. Use the likelihood reduction of an outcome when its mitigative controls make the outcome less likely to follow the event, rather than less severe.

A Risk takes the level of its worst event and outcome pair, unless its aggregation collapses its events into one pair. See [Rate a Risk](/docs/sightline/usage/lamina/risks#rate-a-risk).

## Zones

A zone id on a cause, outcome, exposure, vector or control names a zone in a Tetra model. A cause passes its zones to its preventative controls, and an outcome passes its zones to its mitigative controls. A control that sets its own zones uses those instead. The zones decide which zone assessments colour the element badges of a control. See [Zone ids name Tetra zones](/docs/sightline/usage/lamina/authoring-controls#zone-ids-name-tetra-zones).

## Choosing between them

| You want to record | Use |
|---|---|
| The point where control is lost | An event |
| A way the event can start | A cause |
| A result of the event | An outcome |
| Who or what triggers a cause | The actor of the cause |
| The route a cause takes to the target | The vector of the cause |
| What a cause can reach, and where | The exposure of the cause |
| A barrier whose effectiveness you rate | An assessed control |
| A baseline barrier that is either in place or not | An inherent control |
| A barrier on an outcome that counts only under a condition | A conditional modifier |
| A barrier that stops a cause | An assessed control of type Preventative, on the cause |
| A barrier that limits an outcome | An assessed control of type Mitigative, on the outcome |
| A barrier that only detects the cause or the event | An assessed control of type Detective |
| How likely a cause is with no controls | The inherent likelihood of the cause |
| How likely a cause is with today's controls | The current likelihood of the cause |
| How severe an outcome is before its controls act | The inherent consequence of the outcome |
| How severe an outcome is after its controls act | The residual consequence of the outcome |
| A set of paths rated together | A Risk |

Repeated bodies belong in a template. See [Reuse comes from a template](/docs/sightline/usage/lamina/authoring-controls#reuse-comes-from-a-template).
