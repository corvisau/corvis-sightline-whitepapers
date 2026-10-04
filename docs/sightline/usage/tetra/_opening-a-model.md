The **Sightline** view in the activity bar is the main way to open a model.

## Open from the Models tree

1. Select **Sightline** in the activity bar.
2. Find the model in the **Tetra** group of the **Models** tree. The tree lists the `*.tetra.yaml` manifests in the workspace.
3. Select the manifest. Tetra opens it in the diagram editor.

The diagram editor registers as an optional editor for `*.tetra.yaml` files. To open a manifest in another way, use one of these:

- The command **Tetra: Open Block Diagram**.
- The command **Reopen Editor With…** on a `*.tetra.yaml` file that is already open.

To create a model, run **Tetra: New Model**.

## Manifest file name

A Tetra manifest uses the extension `.tetra.yaml`.

## Edit an entity

Edit and delete an entity from the canvas or the Tabular Editor.

- **Canvas**: right-click a node, then choose **Edit** or **Delete**.
- **Tabular Editor**: right-click a row, then choose **Edit**, **View Code** or **Delete**. See [Bulk edit](/docs/sightline/usage/shared/bulk-edit) for the row context menu.
- **Tabular Editor, empty space**: right-click the header, a group row or the space below the last row. The menu matches the empty canvas: **Edit…** and **View Code** for the model's top-level container, then an **Add** entry for each entity type. Input fields keep the browser menu.

The Model Tree lists and reveals entities; it has no edit actions of its own. See [Tree browser panel](/docs/sightline/usage/shared/tree-browser-panel).

## Related

- [Tree browser panel](/docs/sightline/usage/shared/tree-browser-panel)
- [Templates](/docs/sightline/usage/tetra/templates)
