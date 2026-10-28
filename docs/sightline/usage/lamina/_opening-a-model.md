The **Sightline** view in the activity bar is the main way to open a model.

## Open from the Models tree

1. Select **Sightline** in the activity bar.
2. Find the model in the **Models** tree. The tree shows the workspace as folders, and a Lamina model is marked **Lamina**. It lists every Lamina model in the workspace.
3. Select the model. Lamina opens it in the diagram editor.

To open a model in another way, use one of these:

- The command **Lamina: Open Bow-Tie Diagram**.
- The command **Reopen Editor With…** on a Lamina model file that is already open.

Each of these opens only a model that the Models tree lists. For any other file, Lamina shows a message that names the file and opens nothing.

To create a model, select **New…** on the folder that should hold it and choose **Lamina: New Bow-Tie Model**. The form shows that folder and cannot change it. The form asks for a Name and an Id. The Name is the label that the **Models** tree and the diagram show, and it can hold any text on one line. The Id names the model, for example `my-risk`. The Id fills in from the Name as you type, until you edit it. It has at most 64 characters: lowercase letters, digits and single hyphens. It must differ from the id of every other model in the workspace, and it does not change after Lamina creates the model. Lamina writes the model's files directly in the folder and makes no folder of its own. Running **Lamina: New Bow-Tie Model** from the Command Palette creates the model in the workspace root.

Each file of the new model is named after its Id, for example `my-risk.bowtie.yaml` and `my-risk.causes.yaml`. The folder can already hold a Tetra model or a Metron package. Lamina writes nothing and shows an error when the folder holds another Lamina model or a file that the model would write. Sightline cannot move or delete a model whose folder holds another model, so give each model its own folder when you expect to move it. See [Rename, move and delete models and folders](/docs/sightline/usage/shared/manage-models-and-folders#when-sightline-refuses).

## Model id

The Models tree lists a model only when it has an id, and a model opens only when the tree lists it. A model with no id does not open. The message names the file. It offers a command that adds an id to every model in its folder.

## Related

- [Risks](/docs/sightline/usage/lamina/risks)
- [Authoring controls](/docs/sightline/usage/lamina/authoring-controls)
