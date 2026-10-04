When you select objects on the bow-tie canvas, Lamina dims every object that the
selection is not connected to. An optional Pulse Glow runs travelling pulses
along the lines between the objects that stay lit.

## What stays lit

Lamina lights the union of the connections of every selected object. A connection
follows the risk path in both directions.

| Selected | Lit |
|---|---|
| A cause | The cause, its controls, its events, and their outcomes with their controls |
| An event | Every cause that feeds it with its controls, and every outcome it feeds with its controls |
| An outcome | The outcome and its controls, the events that feed it, and their causes with their controls |
| A cause control | The control, its cause, and that cause's events and outcomes with their controls. Sibling controls on the cause are dimmed |
| An outcome control | The control, its outcome, the events that feed it, and their causes |

Select several objects with Shift-click to light the union of their connections.
Click the empty canvas to clear the selection and the dimming.

Annotations follow the object they link to. An annotation dims when none of its
linked objects is lit. The risk matrix and overlay regions never dim. A selected
annotation alone does not dim anything.

A line stays lit when both of its ends are lit. Every other line dims with the
objects it joins.

## View menu options

Two rows in the View menu control the behaviour.

| Row | Options | Default | Effect |
|---|---|---|---|
| Focus | Shade, Off | Shade | Dims unrelated objects, annotations and lines to 30% opacity while something is selected |
| Pulse Glow | On, Off | Off | Runs pulses along each lit line, from the cause towards the event and from the event towards the outcome |

The two rows are independent. Pulse Glow works with Focus set to Off, in which case
nothing dims and the lit lines pulse.

Neither option is saved with a data slice. Both return to their defaults when the
panel reloads, and a PNG export or headless render, which has no selection, is
unaffected.

## Reduced motion

With the operating system's reduced-motion setting on, the pulses stop moving and
the lit lines keep a static dashed glow.

## Pulse Glow in Tetra

Tetra's Pulse Glow view option draws the same effect on a selected channel or
flow. See [Data slices](/docs/sightline/usage/lamina/data-slices) for the options a slice does save.
