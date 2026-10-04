Lamina and Tetra open each other at the object a Tetra Vector links. A cause that links a Tetra Vector opens the Tetra model on its channel or flow. A channel or flow lists the Lamina causes that cover it and opens one. [Tetra Vector links](/docs/sightline/usage/lamina/tetra-vector-links) explains how a cause links a vector.

## From a cause to Tetra

The props drawer of a cause has a Tetra Vector section when the cause's vector holds a Tetra Vector link. The section shows:

- The kind and id of the channel or flow
- The zone the vector enters
- The id of the Tetra model, when the link names one

Select **Open in Tetra**. Tetra opens the model, selects the channel or flow, opens its props drawer and centres the diagram on it. A model that is already open keeps its tab.

The kind, the id and the zone come from the snapshot stored beside the link. A link with no snapshot shows the Tetra Vector key in their place, and the button still works.

### Finding the Tetra model

Lamina looks for the model in the folder of the `*.bowtie.yaml` manifest and in the folders below it. The result depends on the link.

| Link | Result |
|---|---|
| Names a model, and the model holds the key | Tetra opens that model on the channel or flow. |
| Names no model, and one model holds the key | Tetra opens that model on the channel or flow. |
| Names no model, and several models hold the key | A list of the models appears. Choose one. |
| Names a model that no longer holds the key | A message says the Tetra Vector no longer exists. Tetra opens the model on the channel or flow that the snapshot names, if it still exists. |
| Names no model, and no model holds the key | A warning says that no Tetra model has the Tetra Vector. |
| Names a model outside the manifest's folder | Tetra opens that model on the channel or flow that the snapshot names, if the workspace holds the model. |

## From a channel or flow to its causes

The props drawer of a channel or flow has a Causes section. It lists each Lamina cause that covers the channel or flow, with the cause id, its label and the name of its Lamina model. **Open** opens that Lamina model with the cause selected and its props drawer open. A channel or flow with no cause shows the text "No Lamina cause covers this".

A cause covers a channel or flow when its vector links one of the Tetra Vectors of that channel or flow. A link that names a Tetra model counts for that model only. A link with no model counts for any model that holds the key.

Tetra reads every `*.bowtie.yaml` manifest in the workspace, wherever the author filed it, and the files each one lists. The link stays on the cause. Tetra works out the list on each load and stores nothing, so **Refresh Diagram** and an external file change both update it.

### Cause badges

A badge on the line of a channel or flow shows how many causes cover it. The tooltip reads "2 Lamina causes cover this", for example. Select the badge to open the props drawer of the channel or flow.

The badge sits to the right of the midpoint of the line. The findings badge sits above the midpoint and the coverage badge below it.

The **Cause Badges** row in the View menu turns the badges on or off. The default is on. A data slice saves the setting with its other View options. A model that no cause links shows no badge either way. PNG export draws no cause badge.

## Per-app differences

| | Lamina | Tetra |
|---|---|---|
| Where | Tetra Vector section in the props drawer of a cause | Causes section in the props drawer of a channel or flow, and a badge on the line |
| Action | **Open in Tetra** | **Open**, and the badge |
| Can be turned off | No | Badges only, in the View menu |
| Reads | The Tetra models under the manifest's folder | Every Lamina model in the workspace |
