A zone's **Threat Potential** is how likely a threat actor is to be present in the
zone. It is a whole number from 0 to 4:

| Value | Typical zone |
| --- | --- |
| 0 | A zone where a threat actor is very unlikely to be present |
| 1 | A well-controlled operational zone |
| 2 | A zone with constrained access, such as a privileged-access tier |
| 3 | Corporate IT with routine internet use |
| 4 | The public internet, a carrier network or a personal device |

For when to use a zone and how it differs from a group, see [Tetra entities](/docs/sightline/methodology/tetra-entities#zones-and-groups).

Threat Potential is not the SL-T. SL-T is the strength of controls the zone needs.
Threat Potential describes who is likely to be in the zone.

Lamina reads a zone's Threat Potential and suggests a likelihood for each cause
whose source zone it is. The cause keeps its own likelihood until you choose to
use the suggested one.

## Setting a Zone's Threat Potential

1. Select the zone to open its props drawer.
2. Enter a value from 0 to 4 in **Threat Potential**.
3. Save your changes.

The zone's header shows the value as a blue **TP** pill, for example **TP 3**. It
sits left of the red SL-T pill, and the draw.io export carries the same pills. A
zone with no value shows no pill. The header bar has the same height whether it
shows two pills, one pill or none.

A zone with no value suggests no likelihood. A zone that takes its fields from a
template must set **Threat Potential** on the zone itself, because Lamina does
not read the template's value.
