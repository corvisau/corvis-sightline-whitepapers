Tetra writes a picture of a diagram to the `reports/` directory of the model directory.

## In the editor

**Export PNG** and **Export all Dataslices as PNG** render in a separate, offscreen view that the extension drives. The diagram on screen is never touched. Because VS Code has no fully hidden view, a render tab briefly appears beside the editor and closes when the batch completes.

- **Export PNG** reproduces the current on-screen view exactly. The renderer receives the live model, the collapse state and the view options.
- **Export all Dataslices as PNG** loads each data slice from disk and renders it. Each picture shows the saved view of its slice, including the saved collapse state and view options. Unsaved changes on screen do not appear. The entity inventory report reads the model from disk in the same way.

## Related

- [PNG export in Lamina](/docs/sightline/usage/lamina/png-export) works in the same way.
- [Data slice filtering](/docs/sightline/usage/tetra/data-slice-filtering)
