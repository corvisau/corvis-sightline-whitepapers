Tetra paints Metron's assessment coverage onto the diagram, so an architect can
see how far each entity has been assessed without opening Metron. The overlay
sits beside the findings badges described in [Findings overlay](/docs/sightline/usage/tetra/findings-overlay). See
the Metron pages for how assessments are authored.

## Switching it on

The overlay is off by default. Run **Sightline: Toggle Coverage Overlay (this
diagram)** from the command palette, or use the toolbar button beside **View
Findings on Diagram**. Run the command again to hide the overlay.

The overlay reads Metron data, which needs a licence that names Metron data. See [Licensing](/docs/sightline/usage/shared/licensing).

The switch belongs to one diagram tab. Two open Tetra diagrams toggle
independently, and a tab that is closed and reopened starts with the overlay off.

Tetra fetches coverage when the overlay is switched on and again on every model
reload while it stays on. A diagram whose overlay is off makes no coverage request.

During a reload the previous badges stay on the diagram until the new result
arrives.

## The badge

Any Zone, Group, Device, Network, Information or System node that Metron has
assessed shows a small badge in its bottom-right corner. Channels and flows show
the same badge on the canvas line, drawn just below the midpoint of the routed
path. A channel that also carries findings shows the findings badge above the
line and the coverage badge below it.

The badge holds a bar and a `done/total` count. The bar has one segment per
state:

| Segment | Meaning |
|---|---|
| Green | Done |
| Amber | In progress |
| Grey | Draft |
| Blue | Done, with a not-applicable outcome. Shown only when present |

The count is the number of done pairs over all pairs for the entity. The
accessible name of the badge spells out every bucket, for example `3 of 5
assessed, 1 in progress, 1 draft`.

An entity that Metron has not assessed gets no badge. Overlay regions never get
one, because an assessment cannot bind to an overlay.

## The popup

Hover over a badge to open a read-only popup:

- The done, in-progress and draft counts, the not-applicable count when there is
  one, and the total.
- A revalidation line when any done pair has drifted from the current
  requirement or entity version.
- The outcome distribution of the done pairs. Outcomes with a count of zero are
  left out.
- One row per package that assesses the entity, with its own `done/total`.

To change anything, open the entity's assessment in Metron.

## What the numbers mean

The figures follow Metron's rules and match its worklist and the
[Portfolio Roll-Up](/docs/sightline/usage/metron/portfolio-rollup). Tetra resolves every
`*.assessment.yaml` binding for the diagram's model against the current model.
An entity that came into a requirement's scope since the last save counts as
draft. A pair whose requirement has been removed is left out. The read writes
nothing.

When several packages assess the same entity, the badge adds their counts
together, and the popup lists each package. Each binding uses the package
version it is pinned to.

## When nothing shows

If the overlay is on but no badge appears, a notice at the bottom-left of the
canvas gives the reason:

| Notice | Cause |
|---|---|
| `Coverage unavailable: <reason>` | The licence does not name Metron data. The reason says so |
| `Coverage could not be loaded: <reason>` | The read failed, for example because no workspace folder is open |

With no notice, the model has no assessment bindings, or none of its entities
have pairs yet.
