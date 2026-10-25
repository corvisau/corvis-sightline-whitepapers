A client risk matrix rates likelihood on a scale of keys, such as A to E. This page covers what the scale values mean and how to combine more than one cause's likelihood into an aggregated value.

Editing the scale itself (its keys, labels, probabilities and thresholds) is not available in the UI. Lamina shows the scale wherever a likelihood appears.

---

## What the scale values mean

A scale does not have to use a 0 to 1 probability. A value only needs to be non-negative. Use whichever convention matches how your organisation rates likelihood:

- **Probability**, from 0 to 1. The built-in A to E default scale runs from 0.0032 for A to 0.89 for E.
- **Per-annum frequency**, in events a year and unbounded above. An example is 0.05, 0.5, 2, 5, 10 and 20 for six keys.

The probability of a key is its representative point value, such as the midpoint of a band. The threshold of a key is the upper bound of its bucket. Lamina uses the thresholds to convert an aggregated value back to a key. Both accept the same range.

## Combine more than one cause

An aggregation can combine more than one cause's likelihood. The `simple-damped-union`, `exposure-grouped` and `exposure-vector-grouped` modes do this. The `formulaVersion` field picks the formula.

| `formulaVersion` | Behaviour |
|---|---|
| `1` (default) | Inputs above 1 are clamped to 1 first, so an unbounded (frequency) scale cannot send the combined value negative or off-scale. This keeps the output safe, but it is not a mathematically correct combination for a frequency scale. |
| `2` | Poisson rate-to-probability transform. Exact for a per-annum-frequency scale. It reduces to plain rate addition when a mode's damping factor is 1, and gives a principled discounted value when damping is below 1. |
| absent | The aggregation uses the formula that the model's risk matrix sets. When the model sets none, it uses the legacy damped union. |

Set `formulaVersion: 2` when your likelihood scale is a per-annum frequency and the aggregation uses any mode other than `max`. The `max` mode takes the highest single cause and combines nothing, so the formula has no effect on it.

## Set the formula for the model default

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
