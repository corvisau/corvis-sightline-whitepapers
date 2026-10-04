Lamina, Tetra and Metron install as part of the **Sightline** extension. Each
app has no separate extension. The extension adds one **Sightline** view to the
activity bar. It holds risk models as bow-tie diagrams (Lamina), architecture
models as block diagrams (Tetra) and compliance assessment (Metron).

Metron assesses a Tetra model, so you need Tetra model files when you use
Metron. See [Cross-app workflows](/docs/sightline/architecture/cross-app-workflows).

## Install a VSIX file

Sightline ships as one `.vsix` file. To install a file:

1. Open the **Extensions** view in VS Code.
2. Select the **…** menu at the top of the view, then **Install from VSIX…**.
3. Select the `.vsix` file.

You can also run this command in a terminal. Replace `<file>` with the name of the file:

```bash
code --install-extension <file>.vsix
```

Reload VS Code when it asks you to.

## Check the install

1. Select the **Sightline** activity-bar view. It shows a combined **Models** tree with **Lamina**, **Tetra** and **Metron** groups.
2. Open a model. See [Open a Tetra model](/docs/sightline/usage/tetra/opening-a-model) or [Open a Lamina model](/docs/sightline/usage/lamina/opening-a-model).

## Next step

[Quickstart](/docs/sightline/usage/quickstart)
