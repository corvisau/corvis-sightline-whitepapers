Export PNG on the diagram toolbar writes a picture of the current bow-tie to the
model's `outputs/` folder. A dialog collects the filename and whether to frame
the diagram to a fixed aspect ratio. The diagram draws one Risk. Export PNG and
Export Draw.io are disabled until you pick a Risk.

## The export renders separately from what is on screen

The diagram is re-rendered in a dedicated, offscreen view and captured there.
The picture therefore does not depend on the state of the editor when you press
the button. Zoom level, scroll position, a hovered node, a selected node and an
open dialog all stay out of the image.

The export still shows the model and view settings you are working with. Hidden
elements stay hidden, the picture shows the Risk that you picked, and display
options such as compact mode are honoured.

A progress notification appears while the export runs, and you can cancel it.
The editor stays usable throughout.

## What the picture always contains

Three rules apply to every exported PNG, whatever the editor looks like:

- The risk matrix is excluded. Export the matrix separately if you need it.
- The image renders in the light theme, even when VS Code uses a dark one, so
  the picture stays legible in documents and slides.
- The whole diagram is framed. With the fixed-ratio option the diagram is
  centred and the short side is padded.

## Exporting every Risk

Export, then All Risks (PNG), writes one picture for each Risk of the model. Each picture is rendered in the same offscreen view as a single export, one after another, under one progress notification. Cancel stops the batch after the picture in progress; the closing message reports how many pictures were written before the cancel.

Each picture shows the Risk as the Diagram draws it by default: the causes and outcomes on the paths of the Risk, around one aggregated event that carries the rating of the Risk. The files land in `outputs/` and are named `<model>-<Risk name>.png`. Characters that a file name cannot hold become `-`. When two Risks share a name, the second file takes its Risk id as a suffix.

Every picture in a batch is framed at 24:11 and carries a physical size of 24 × 11 cm, so PowerPoint inserts it at that size. The single export keeps the framing chosen in its dialog.

The batch reads the saved model. It skips unsaved edits, so save first when the pictures must show an unsaved Risk. A Risk with no path produces no file, and the closing message counts it as skipped. A model that defines no Risks writes no file, and the message says that there are no Risks to export.

## Draw.io export is unaffected

Export Draw.io builds its file from the diagram's structure rather than from a
picture of the canvas, so the editor's state does not affect it.

## Related

- Tetra exports PNGs the same way, including every data slice at once with the
  same 24 × 11 cm framing: see [PNG export in Tetra](/docs/sightline/usage/tetra/png-export).
- Risks and the Diagram: see [Risks](/docs/sightline/usage/lamina/risks#pick-a-risk).
