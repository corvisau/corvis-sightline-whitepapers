Export PNG on the diagram toolbar writes a picture of the current bow-tie to the
model's `outputs/` folder. A dialog collects the filename and whether to frame
the diagram to a fixed aspect ratio.

## The export renders separately from what is on screen

The diagram is re-rendered in a dedicated, offscreen view and captured there.
The picture therefore does not depend on the state of the editor when you press
the button. Zoom level, scroll position, a hovered node, a selected node and an
open dialog all stay out of the image.

The export still shows the model and view settings you are working with. Hidden
elements stay hidden, the active aggregation is applied, and display options
such as compact mode are honoured.

A progress notification appears while the export runs, and you can cancel it.
The editor stays usable throughout.

## What the picture always contains

Three rules apply to every exported PNG, whatever the editor looks like:

- The risk matrix is excluded. Export the matrix separately if you need it.
- The image renders in the light theme, even when VS Code uses a dark one, so
  the picture stays legible in documents and slides.
- The whole diagram is framed. With the fixed-ratio option the diagram is
  centred and the short side is padded.

## Exporting every data slice

Export, then All slices (PNG), writes one picture for each saved data slice. The
item appears only when the model has at least one data slice.

Each picture is rendered in the same offscreen view as a single export, one after
another, under one progress notification. Cancel stops the batch after the slice
in progress; the closing message reports how many pictures were written before
the cancel.

The files land in `outputs/` and are named `<model>-<slice name>.png`. Characters
that a file name cannot hold become `-`. When two slices share a name, the second
file takes its slice id as a suffix.

Every picture in a batch is framed at 24:11 and carries a physical size of
24 × 11 cm, so PowerPoint inserts it at that size. The single export keeps the
framing chosen in its dialog.

The batch reads the model and the slices from disk. It skips unsaved edits and
any unsaved view change on a slice (compact, tags, elements, outcome sort or
annotation display). Save first when the pictures must show them. Each slice
applies its own filter and aggregation, and the batch does not use the slice
currently open in the editor.

An empty slice produces no file. The closing message counts it as skipped.

## Draw.io export is unaffected

Export Draw.io builds its file from the diagram's structure rather than from a
picture of the canvas, so the editor's state does not affect it.

## Related

- Tetra exports PNGs the same way, including every data slice at once with the
  same 24 × 11 cm framing: see [PNG export in Tetra](/docs/sightline/usage/tetra/png-export).
- Data slices: see [Lamina Data Slices](/docs/sightline/usage/lamina/data-slices).
