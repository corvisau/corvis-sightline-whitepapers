The **Sightline** view in the activity bar is the main way to open a model.

## Open from the Models Tree

1. Select **Sightline** in the activity bar.
2. Find the model in the **Models** tree. The tree shows the workspace as folders, and a Lamina model is marked **Lamina**. It lists every Lamina model in the workspace.
3. Select the model. Lamina opens it in the diagram editor.

To open a model in another way, use one of these:

- The command **Lamina: Open Bow-Tie Diagram**.
- The command **Reopen Editor With…** on a Lamina model file that is already open.

Each of these opens only a model that the Models tree lists. For any other file, Lamina shows a message that names the file and opens nothing.

To create a model, select **New…** on the folder that should hold it and choose **Lamina: New Bow-Tie Model**. The form shows that folder and cannot change it. The form asks for a Name and an Id. The Name is the label that the **Models** tree and the diagram show, and it can hold any text on one line. The Id names the model, for example `my-risk`. The Id fills in from the Name as you type, until you edit it. It has at most 64 characters: lowercase letters, digits and single hyphens. It must differ from the id of every other model in the workspace, and it does not change after Lamina creates the model. Lamina writes the model's files into a folder of its own inside the folder you chose. That folder is named after the Id with the ending `.sightline`, for example `my-risk.sightline`. The **Models** tree shows the model in the folder you chose, not in its own folder. Running **Lamina: New Bow-Tie Model** from the Command Palette creates the model in the workspace root.

Each file of the new model is named after its Id and sits in that folder, for example `my-risk.sightline/my-risk.bowtie.yaml` and `my-risk.sightline/my-risk.causes.yaml`. The folder you chose can already hold any other model, including another Lamina model. Lamina writes nothing and shows an error when it already holds a folder of that name. Sightline moves and deletes the model folder as one unit. A model that you created before this layout keeps its files directly in its folder, and Sightline cannot move or delete such a model while its folder holds another model. See [Rename, move and delete models and folders](/docs/sightline/usage/shared/manage-models-and-folders#refused-model-and-folder-operations). To move such a model into its own folder, see [Model folders](/docs/sightline/upgrade/compatibility#model-folders).

## View Code in the Web Shell

In the web shell, **View Code** and **View Code of Target** in a context menu open the shell's YAML viewer at the element's `id:` line. **View Code** in the selection drawer does the same. The viewer is read-only and offers **Download file**. A Lamina page opened outside the web shell downloads the source file directly.

## Model ID

The Models tree lists a model only when it has an id, and a model opens only when the tree lists it. A model with no id does not open. The message names the file. It offers a command that adds an id to every model in its folder.

## Related

- [Risks](/docs/sightline/usage/lamina/risks)
- [Authoring controls](/docs/sightline/usage/lamina/authoring-controls)
