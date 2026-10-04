A client risk-matrix definition (`manifest.riskMatrixDef`) declares `likelihood.probability`/`likelihood.thresholds` per key. This doc covers what those values mean and how to combine more than one cause's likelihood into an aggregated value.

---

## `probability` / `thresholds` value range

Despite the field name, `likelihood.probability` (and the optional `likelihood.thresholds`) is not required to be a `0-1` probability. It only needs to be non-negative. Use whichever convention matches how your organisation rates likelihood:

- **Probability** (`0-1`, e.g. the built-in A-E default scale: `A: 0.0032` ... `E: 0.89`).
- **Per-annum frequency** (events/year, unbounded above, e.g. `a: 0.05, b: 0.5, c: 2, d: 5, e: 10, f: 20`).

`probability` is the representative point value per key (e.g. a band's midpoint); `thresholds` is the bucket's upper bound, used to convert an aggregated value back to a key. Both accept the same range.

## `likelihood.formulaVersion` — combining more than one cause

When a view's aggregation config combines more than one cause's likelihood (`simple-damped-union`, `exposure-grouped`, `exposure-vector-grouped`), the def's `likelihood.formulaVersion` picks the combination formula:

| Value | Behaviour |
|---|---|
| absent or `1` (default) | Legacy damped-union formula. Inputs above `1` are clamped to `1` first so an unbounded (frequency) scale can't send the combined value negative or otherwise off-scale. This keeps the output safe, but it is not a mathematically correct combination for a frequency scale. |
| `2` | Poisson rate-to-probability transform: exact for a per-annum-frequency scale (reduces to plain rate addition when a mode's damping factor is `1`, and a principled discounted value when damping is `< 1`). |

Set `formulaVersion: 2` if your def's likelihood scale is a per-annum frequency and you use any aggregation mode other than `max`. `max` mode (highest single cause wins, no arithmetic combination) is unaffected by this setting either way.

```yaml
likelihood:
  keys: [a, b, c, d, e, f]
  labels: { a: Rare, b: Unlikely, c: Possible, d: Likely, e: Frequent, f: Continuous }
  probability: { a: 0.05, b: 0.5, c: 2, d: 5, e: 10, f: 20 }
  thresholds: { a: 0.1, b: 1, c: 3, d: 8, e: 15, f: 20 }
  formulaVersion: 2
```

## Per-aggregation override

An aggregation's own Likelihood config can override the def-level `formulaVersion` for that aggregation only. The aggregation's own setting wins. Otherwise the definition's value applies, and otherwise `1`. Set it from the Aggregation Settings panel's Likelihood section. The **Combination formula** field sits next to `dampingFactor`/`withinDamping`/`acrossDamping` and shows for every mode except `max`, which never combines more than one cause.

The field starts unset. If the def's `formulaVersion` changes later, an aggregation that never set its own value picks up the new def value on the next load. Pick an explicit value only when this particular aggregation needs a formula different from the rest of the model.
