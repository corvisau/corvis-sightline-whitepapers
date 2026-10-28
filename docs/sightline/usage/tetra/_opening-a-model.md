The **Sightline** view in the activity bar is the main way to open a model.

## Open from the Models tree

1. Select **Sightline** in the activity bar.
2. Find the model in the **Models** tree. The tree shows the workspace as folders, and a Tetra model is marked **Tetra**.
3. Select the model. Tetra opens it in the diagram editor.

To open a model in another way, use one of these:

- The command **Tetra: Open Block Diagram**.
- The command **Reopen Editor With…** on a model file that is already open, then the choice **Tetra Block Diagram**.

To create a model, select **New…** on the folder that should hold it and choose **Tetra: New Model**. The form shows that folder and cannot change it. The form asks for a Name and an Id. The Name is the label that the **Models** tree and the diagram show, and it can hold any text on one line.

The Id identifies the model, for example `my-plant`. The Id fills in from the Name as you type, until you edit it. It has at most 64 characters: lowercase letters, digits and single hyphens. It must differ from the id of every other model in the workspace, and it does not change after Tetra creates the model. Tetra creates the model directly in the folder and makes no folder of its own. Running **Tetra: New Model** from the Command Palette creates the model in the workspace root.

Each file of the new model is named after its Id, for example `my-plant.tetra.yaml` and `my-plant.zones.yaml`. The folder can already hold a Lamina model or a Metron package. Tetra creates nothing and shows an error when the folder holds another Tetra model or a file that the new model would use. Sightline cannot move or delete a model whose folder holds another model, so give each model its own folder when you expect to move it. See [Rename, move and delete models and folders](/docs/sightline/usage/shared/manage-models-and-folders#when-sightline-refuses).

## Models outside the workspace

Tetra opens only a model that the Models tree lists, and the tree lists the Tetra models in the workspace folders. A model from outside the workspace folders does not open. Tetra shows a message that names the file. Add the folder that holds the model to the workspace. Then open the model from the tree. A model inside a hidden folder, whose name starts with a dot, or inside `node_modules` is not listed either. Move it to a visible folder.

## Edit an entity

In the browser, select a Tetra model in the sidebar to open its diagram. The diagram and Table use the same editing controls as the extension.

The toolbar opens **Templates**, **Constants** and **Rules** in separate tabs. Your diagram stays open with its zoom, selection and unsaved changes. Opening an editor again focuses its existing tab.

Select **Tools** for these actions:

- **Model tree** opens the entity tree beside the editor.
- **Staged changes journal** lists unsaved edits and offers **Undo** and **Redo**.
- **View findings on diagram** narrows the diagram to entities with authored findings. It is disabled when no finding references the model.
- **Show coverage overlay** displays assessment coverage. Select **Hide coverage overlay** to switch it off.
- **Push layout nudges to base slice** opens the confirmation for moving the active slice's nudges into the slice it lists.

The browser shows the bundled device images and default entity icons. Icon choices from VS Code workspace settings apply in the extension.

Close a tool panel with its close button. Each model tab keeps its own tree, journal and coverage state.

**View Code** in a context menu opens the browser's YAML viewer at the entity's declaration. The viewer is read-only and offers **Download file**. Use the entity, Template Editor or Constants Editor controls to change the model. A Tetra page opened outside the web shell downloads the source file directly.

Edit and delete an entity from the canvas or the Tabular Editor.

- **Canvas**: right-click a node, then choose **Edit** or **Delete**.
- **Tabular Editor**: right-click a row, then choose **Edit**, **View Code** or **Delete**. See [Bulk edit](/docs/sightline/usage/shared/bulk-edit) for the row context menu.
- **Tabular Editor, empty space**: right-click the header, a group row or the space below the last row. The menu matches the empty canvas: **Edit…** and **View Code** for the model's top-level container, then an **Add** entry for each entity type. Input fields keep the browser menu.

The Model Tree lists and reveals entities, and its row toggles choose what the active data slice shows. See [Choosing what a slice shows from the Model Tree](/docs/sightline/usage/tetra/data-slice-filtering#choosing-what-a-slice-shows-from-the-model-tree). It has no entity edit actions of its own. See [Tree browser panel](/docs/sightline/usage/shared/tree-browser-panel).

## Related

- [Tree browser panel](/docs/sightline/usage/shared/tree-browser-panel)
- [Templates](/docs/sightline/usage/tetra/templates)
