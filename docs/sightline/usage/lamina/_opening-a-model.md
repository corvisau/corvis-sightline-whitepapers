The **Sightline** view in the activity bar is the main way to open a model.

## Open from the Models tree

1. Select **Sightline** in the activity bar.
2. Find the model in the **Models** tree. The tree shows the workspace as folders, and a Lamina model is marked **Lamina**. It lists the `*.bowtie.yaml` manifests in the workspace.
3. Select the manifest. Lamina opens it in the diagram editor.

Lamina registers the diagram editor and the Template Editor as optional editors for `*.bowtie.yaml` files. They do not take over every YAML file. To open a manifest in another way, use one of these:

- The command **Sightline: Open Bow-Tie Diagram**.
- The command **Reopen Editor With…** on a `*.bowtie.yaml` file that is already open.

Each of these opens only a manifest that the Models tree lists. For any other file, Lamina shows a message that names the file and opens nothing.

To create a model, select **New…** on the folder that should hold it and choose **Sightline: New Model**. The form shows that folder and cannot change it. Lamina writes the model's files directly in that folder, with the manifest named for the model's name, for example `my-risk.bowtie.yaml`. Lamina makes no folder of its own. The folder must not already hold a model: Lamina writes nothing and shows an error when it holds a Sightline manifest or a file that the model would write. Running **Sightline: New Model** from the Command Palette creates the model in the workspace root.

## Model id

Every manifest needs a `meta.id`:

```yaml
meta:
  id: example-risk-model
```

The Models tree lists a manifest only when it has an id, and a model opens only when the tree lists it. A manifest with no `meta.id` does not open. The message names the file. It gives a command that adds an id to every manifest in its folder, taking the id from the file name. Adding the line by hand has the same effect.

## Manifest file name

A Lamina manifest uses the extension `.bowtie.yaml`. Rename a manifest that uses `.model.yaml`.

Lamina migrates a manifest from an earlier version when it loads. It updates `schemaVersion` from 0 or 1 to 2, and it renames `views` to `dataSlices`.

## Related

- [Data slices](/docs/sightline/usage/lamina/data-slices)
- [Authoring controls](/docs/sightline/usage/lamina/authoring-controls)
