Lamina and Tetra open each other at the object that a record concerns. A cause that links a record opens the Tetra model on its channel or flow. A channel or flow lists the Lamina causes that cover it and opens one. [Record links](/docs/sightline/usage/lamina/finding-links) explains how a cause links a record.

## From a cause to Tetra

The props drawer of a cause has a **Records** section with one block for each link. A block that names a Tetra model has an **Open in Tetra** button.

Select **Open in Tetra**. Tetra opens the model and selects the first channel or flow among the links of the record. It opens the props drawer of that object and centres the diagram on it. A model that is already open keeps its tab.

The result depends on the link.

| Link | Result |
|---|---|
| Names a model, and a link of the record is a channel or flow of that model | Tetra opens that model on the channel or flow. |
| Names a model, and no link is a channel or flow of that model | Tetra opens that model with nothing selected. |
| Names no model | A warning says that the link names no Tetra model to open. |
| Names a model that the workspace does not list | A warning gives the reason that the model did not open. |

## From a channel or flow to its causes

The props drawer of a channel or flow has a Causes section. It lists each Lamina cause that covers the channel or flow, with the cause id, its label and the name of its Lamina model. **Open** opens that Lamina model with the cause selected and its props drawer open. A channel or flow with no cause shows the text "No Lamina cause covers this".

A cause covers a channel or flow when its vector links a record that has the channel or flow among its links. A link that names a Tetra model counts for that model only. A link with no model counts for any model that holds the record.

Tetra reads every `*.bowtie.yaml` manifest in the workspace, wherever the author filed it, and the files each one lists. The link stays on the cause. Tetra works out the list on each load and stores nothing, so **Refresh Diagram** and an external file change both update it.

### Cause badges

A badge on the line of a channel or flow shows how many causes cover it. The tooltip reads "2 Lamina causes cover this", for example. Select the badge to open the props drawer of the channel or flow.

The badge sits to the right of the midpoint of the line. The findings badge sits above the midpoint and the coverage badge below it.

The **Cause Badges** row in the View menu turns the badges on or off. The default is on. A data slice saves the setting with its other View options. A model that no cause links shows no badge either way. PNG export draws no cause badge.

## Per-app differences

| | Lamina | Tetra |
|---|---|---|
| Where | Records section in the props drawer of a cause | Causes section in the props drawer of a channel or flow, and a badge on the line |
| Action | **Open in Tetra** | **Open**, and the badge |
| Can be turned off | No | Badges only, in the View menu |
| Reads | The Tetra models that the workspace lists | Every Lamina model in the workspace |
