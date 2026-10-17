The **Sightline** view in the activity bar is the main way to open a model.

## Open from the Models tree

1. Select **Sightline** in the activity bar.
2. Find the model in the **Models** tree. The tree shows the workspace as folders, and a Tetra model is marked **Tetra**. It lists the `*.tetra.yaml` manifests in the workspace.
3. Select the manifest. Tetra opens it in the diagram editor.

The diagram editor registers as an optional editor for `*.tetra.yaml` files. To open a manifest in another way, use one of these:

- The command **Tetra: Open Block Diagram**.
- The command **Reopen Editor With…** on a `*.tetra.yaml` file that is already open.

To create a model, select **New…** on the folder that should hold it and choose **Tetra: New Model**. The form shows that folder and cannot change it. Tetra writes the model's files directly in that folder, with the manifest named for the model's name, for example `my-plant.tetra.yaml`. Tetra makes no folder of its own. A new model holds the manifest, the entity files `zones.yaml`, `groups.yaml`, `devices.yaml`, `networks.yaml`, `channels.yaml` and `flows.yaml`, and `model.tree.yaml`, which holds the model's top-level container. The folder must not already hold a model: Tetra writes nothing and shows an error when it holds a Sightline manifest or a file that the model would write. Running **Tetra: New Model** from the Command Palette creates the model in the workspace root.

## Models outside the workspace

Tetra opens only a model that the Models tree lists, and the tree lists the `*.tetra.yaml` manifests in the workspace folders. A manifest from outside the workspace folders does not open. Tetra shows a message that names the file. Add the folder that holds the manifest to the workspace, then open the manifest from the tree.

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
