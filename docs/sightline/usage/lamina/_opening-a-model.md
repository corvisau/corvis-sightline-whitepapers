The **Sightline** view in the activity bar is the main way to open a model.

## Open from the Models tree

1. Select **Sightline** in the activity bar.
2. Find the model in the **Lamina** group of the **Models** tree. The tree lists the `*.bowtie.yaml` manifests in the workspace.
3. Select the manifest. Lamina opens it in the diagram editor.

Lamina registers the diagram editor and the Template Editor as optional editors for `*.bowtie.yaml` files. They do not take over every YAML file. To open a manifest in another way, use one of these:

- The command **Sightline: Open Bow-Tie Diagram**.
- The command **Reopen Editor With…** on a `*.bowtie.yaml` file that is already open.

To create a model, run **Sightline: New Model**.

## Manifest file name

A Lamina manifest uses the extension `.bowtie.yaml`. Rename a model from an earlier version that used `.model.yaml`.

Lamina migrates a manifest from an earlier version when it loads. It updates `schemaVersion` from 0 or 1 to 2, and it renames `views` to `dataSlices`.

## Related

- [Data slices](/docs/sightline/usage/lamina/data-slices)
- [Authoring controls](/docs/sightline/usage/lamina/authoring-controls)
