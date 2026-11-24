A client risk matrix rates likelihood on a scale of keys, such as A to E. This page covers what the scale values mean and how to combine more than one cause's likelihood into an aggregated value.

Editing the scale itself (its keys, labels, probabilities and thresholds) is not available in the UI. Lamina shows the scale wherever a likelihood appears.

---

## Likelihood Scale Values

A scale does not have to use a 0 to 1 probability. A value only needs to be non-negative. Use whichever convention matches how your organisation rates likelihood:

- **Probability**, from 0 to 1. The built-in A to E default scale runs from 0.0032 for A to 0.89 for E.
- **Per-annum frequency**, in events a year and unbounded above. An example is 0.05, 0.5, 2, 5, 10 and 20 for six keys.

The probability of a key is its representative point value, such as the midpoint of a band. The threshold of a key is the upper bound of its bucket. Lamina uses the thresholds to convert an aggregated value back to a key. Both accept the same range.

## Combine More than One Cause

An aggregation can combine more than one cause's likelihood. The `simple-damped-union` and `exposure-grouped` modes do this. The `formulaVersion` field picks the formula.

| `formulaVersion` | Behaviour |
|---|---|
| `1` (default) | Inputs above 1 are clamped to 1 first, so an unbounded (frequency) scale cannot send the combined value negative or off-scale. This keeps the output safe, but it is not a mathematically correct combination for a frequency scale. |
| `2` | Poisson rate-to-probability transform. Exact for a per-annum-frequency scale. It reduces to plain rate addition when a mode's damping factor is 1, and gives a principled discounted value when damping is below 1. |
| absent | The aggregation uses the formula that the model's risk matrix sets. When the model sets none, it uses the legacy damped union. |

Set `formulaVersion: 2` when your likelihood scale is a per-annum frequency and the aggregation uses any mode other than `max`. The `max` mode takes the highest single cause and combines nothing, so the formula has no effect on it.

## Set the Formula for the Model Default

Set the formula in the manifest, on the per-event likelihood of the model default aggregation:

```yaml
riskDefaults:
  perEvent:
    likelihood:
      mode: simple-damped-union
      formulaVersion: 2
```

A Risk sets its own formula the same way, under `aggregation.perEvent.likelihood`.

The `max` mode never combines more than one cause, so it ignores the field. An aggregation that sets no `formulaVersion` uses the formula that the model's risk matrix sets, so a later change to the model default applies on the next load. Set a value only when this aggregation needs a formula different from the rest of the model.

---

## Likelihood from a Zone's Threat Potential

A Tetra zone can carry a **Threat Potential** from 0 to 4: how likely a threat
actor is to be present in the zone. The risk matrix maps each value to a
likelihood key, and Lamina uses that key to suggest a likelihood for each cause
whose source zone is that zone.

The built-in A to E scale maps the values like this:

| Threat Potential | Likelihood key |
| --- | --- |
| 0 | A |
| 1 | B |
| 2 | C |
| 3 | D |
| 4 | E |

A model with its own risk matrix declares its own mapping under
`likelihood.threatPotential`, with one entry for each value that should suggest
a likelihood:

```yaml
likelihood:
  keys: [nc, rare, unlikely, possible, likely, certain]
  threatPotential:
    "0": rare
    "1": unlikely
    "2": possible
    "3": likely
    "4": certain
```

A risk matrix without the mapping suggests no likelihood. An entry that names a
key the scale does not have suggests nothing for that value.

## Likelihood from the Source Zone

A cause whose source zone carries a Threat Potential shows a **Zone likelihood**
section in its props drawer. The section names the source zone and its Threat
Potential, and shows the likelihood the risk matrix maps it to. The value is
read-only. The cause's own likelihood does not change.

To use the zone likelihood:

1. Select the cause to open its props drawer.
2. In **Zone likelihood**, select **Use zone likelihood**.
3. Save your changes.

The cause's **Inherent Likelihood** takes the zone's value. When the two already
match, the section says so and offers no action. A later change to the zone's
Threat Potential does not change the cause. The section shows the new value, and
you choose whether to use it.

The source zone is the one that the Tetra flows of the cause start in. When no
likelihood can be suggested, the section says why:

- The cause links no Tetra flow, so it has no source zone.
- The cause links a Tetra record whose accepted data has no `sourceZone`,
  `targetZone` or `pathZones`. The section names the missing fields. Make the
  Tetra rule emit them, then accept the record again.
- The zone has no Threat Potential, or the Tetra models give it different values.
  To settle the second case, name the Tetra model on the record link of the flow.
- The risk matrix maps no likelihood to the zone's Threat Potential.
