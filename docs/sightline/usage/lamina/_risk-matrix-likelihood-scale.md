A client risk matrix rates likelihood on a scale of keys, such as A to E. This page covers what the scale values mean and how to combine more than one cause's likelihood into an aggregated value.

Editing the scale itself (its keys, labels, probabilities and thresholds) is not available in the UI. Lamina shows the scale wherever a likelihood appears.

---

## What the scale values mean

A scale does not have to use a 0 to 1 probability. A value only needs to be non-negative. Use whichever convention matches how your organisation rates likelihood:

- **Probability**, from 0 to 1. The built-in A to E default scale runs from 0.0032 for A to 0.89 for E.
- **Per-annum frequency**, in events a year and unbounded above. An example is 0.05, 0.5, 2, 5, 10 and 20 for six keys.

The probability of a key is its representative point value, such as the midpoint of a band. The threshold of a key is the upper bound of its bucket. Lamina uses the thresholds to convert an aggregated value back to a key. Both accept the same range.

## Combine more than one cause

An aggregation can combine more than one cause's likelihood. The **Simple damped union**, **Exposure grouped** and **Exposure-vector grouped** modes do this. The **Combination formula** field picks the formula.

| Option | Behaviour |
|---|---|
| **Legacy damped union (default)** | Inputs above 1 are clamped to 1 first, so an unbounded (frequency) scale cannot send the combined value negative or off-scale. This keeps the output safe, but it is not a mathematically correct combination for a frequency scale. |
| **Poisson rate-to-probability transform** | Exact for a per-annum-frequency scale. It reduces to plain rate addition when a mode's damping factor is 1, and gives a principled discounted value when damping is below 1. |
| **(use model default)** | The aggregation uses the formula that the model's risk matrix sets. When the model sets none, it uses the legacy damped union. |

Choose **Poisson rate-to-probability transform** when your likelihood scale is a per-annum frequency and the aggregation uses any mode other than **Max**. The **Max** mode takes the highest single cause and combines nothing, so the formula has no effect on it.

## Set the formula for one aggregation

1. In the diagram toolbar, select **Model Properties**. In the **Aggregation** picker at the top of the drawer, select an aggregation.
2. Select the edit button beside the picker. To start a new aggregation, select the add button instead.
3. In the **Edit Aggregation** dialog, expand **Likelihood aggregation**. Switch on **Enable**.
4. In **Mode**, choose a mode other than **Max**.
5. In **Combination formula**, choose an option.
6. Select **Save**.

The **Combination formula** field does not show for the **Max** mode, which never combines more than one cause. The field starts on **(use model default)**. If the model's default changes later, an aggregation that kept that option picks up the new default on the next load. Choose an explicit option only when this aggregation needs a formula different from the rest of the model.
