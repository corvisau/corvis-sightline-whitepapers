A cause in Lamina can link its vector to a Tetra Vector. Tetra derives a Tetra Vector from the architecture: it is a channel or flow that enters a zone from a different zone. Lamina stores a snapshot of the Tetra Vector beside the link. When the model opens, Lamina compares the snapshot with the live Tetra Vector and reports a difference in the Problems panel.

A vector without a link is a hand-written Vector. Lamina does not check it.

The props drawer of a cause also has an **Open in Tetra** button for a linked vector. See [Jump between a cause and its Tetra channel or flow](/docs/sightline/usage/shared/tetra-vector-jump).

## What a linked vector holds

A linked vector carries two fields, both inside the cause's `vector`.

| Field | Content |
|---|---|
| `links` | One link with target kind `entity` and relationship `vector`. The target is the Tetra Vector key, and the instance is the id of the Tetra model. |
| `tetraSnapshot` | The label, the two end nodes, the zone entered, the zone at the other end and the rules that emit the vector, and the likelihood and suggested controls when a rule sets them, plus a hash of them. |

The key is the channel or flow id, then `>`, then the id of the zone it enters. A bidirectional channel gives two Tetra Vectors, one for each zone.

```yaml
causes:
  - id: C001
    label: Remote access to the control zone
    vector:
      id: V001
      label: Example vendor VPN
      links:
        - uriType: entity
          linkType: vector
          instance: example-ot-model
          identifier: ch-vendor-vpn>z-control
      tetraSnapshot:
        v: 1
        hash: 1f3a9c0b7d2e4
        label: Example vendor VPN
        relationKind: channel
        relationId: ch-vendor-vpn
        nodeA: dev-vendor-gateway
        nodeB: dev-control-switch
        zoneId: z-control
        otherZoneId: z-vendor
        ruleIds: []
```

## The three states

Lamina checks each linked vector against the Tetra models in the folder of the `*.bowtie.yaml` manifest, and in the folders below it.

| State | Meaning | Problems panel |
|---|---|---|
| Current | The live Tetra Vector matches the snapshot. | No entry. |
| Changed | The Tetra Vector exists and a field differs. | A warning that lists each field with its old and new value. |
| Gone | Tetra no longer derives the key, or the Tetra model is not found. | A warning that names the cause and the key. |

A Tetra Vector is gone when its channel or flow is removed, when it no longer crosses into the zone, or when no vector rule emits it. A change in the order of the rules does not change the state.

The check runs when a manifest opens and when a YAML file under its folder is saved. A model with no Tetra manifest gets no warnings, and its vectors stay hand-written.

A link with no snapshot, and a snapshot that does not match the schema, are not checked.

## Uncovered Tetra Vectors

The Tetra Vectors section in the Sightline sidebar lists the Tetra Vectors that no cause links. It shows while a Lamina model is open, and it starts collapsed.

Two manifest fields set the scope of the section.

| Field | Content |
|---|---|
| `tetraZones` | The ids of the Tetra zones in scope. |
| `tetraModel` | The id of the one Tetra model to scan. Without it, the section scans every Tetra model in the folder of the manifest and the folders below it. |

```yaml
files:
  - causes.yaml
tetraModel: example-ot-model
tetraZones:
  - z-control
  - z-vendor
```

### Set the scope in the Model Properties drawer

The Model Properties drawer edits both fields, so the manifest needs no hand edit. Open it in one of two ways:

1. Select **Model** in the toolbar.
2. Right-click an empty part of the canvas or of the Tabular Editor, then choose **Edit…**.

Choose a Tetra model, then select the zones in scope. The zone list shows the zones that the chosen model defines. While no model is chosen, it shows the zones of every Tetra model in the folder.

Save the model to write `tetraModel` and `tetraZones` to the manifest. The section refreshes after the save.

A declared zone that the chosen model does not define stays selected, with the note "not defined in this Tetra model". Clear its checkbox to remove it. Choose **(any model)** and clear every zone to remove both fields from the manifest.

The section lists one node for each declared zone that has an uncovered Tetra Vector. Each row under a node is one Tetra Vector that enters that zone. A row shows the label, the kind (channel or flow) and the zone at the other end. The tooltip shows the key, the model, the two end nodes and the rules that emit the vector.

A cause covers a Tetra Vector when its vector holds a link to the key. A link that names a Tetra model covers the vector in that model only. A link with no model covers the key in any model. The snapshot does not matter: a changed or gone vector is still a linked vector, and the Problems panel reports it.

The section follows the focused Lamina model. It refreshes when the focus moves to another Lamina model and when a YAML file under the folder of the manifest is saved.

When the section has no rows, a message above the list says why.

| Message | Cause |
|---|---|
| Open a Lamina model to list its uncovered Tetra Vectors. | No Lamina model is open. |
| No Tetra model under this model's folder. | The folder of the manifest, and the folders below it, hold no Tetra model. |
| Tetra model example-ot-model is not under this model's folder. | `tetraModel` names a Tetra model that the folder of the manifest, and the folders below it, do not hold, or one that failed to load. The id changes with the manifest. |
| Declare tetraZones in the model manifest to list uncovered Tetra Vectors. | The manifest has no `tetraZones`, or the list is empty. Select zones in the Model Properties drawer. |
| All 3 Tetra Vectors in scope have a cause. | Every Tetra Vector in the declared zones has a link. The count changes with the model. |
| No Tetra Vectors enter the declared zones. | No channel or flow enters a declared zone. |
| Could not read this model's Tetra Vectors. | The scan failed. Save the manifest again to retry. |

A declared zone that no scanned Tetra model defines shows as a node with the note "not defined in any Tetra model". Check the spelling of the id. When `tetraModel` is set, only the zones of that model count.

### Cover a listed Tetra Vector

Right-click a row in the Tetra Vectors section to cover its vector. The menu has two items.

| Menu item | Result |
|---|---|
| Create cause from vector | The editor of the model opens the new cause drawer. The drawer holds the label of the vector, the draft state, and the vector link with its snapshot. |
| Link to existing cause… | A picker asks which cause takes the link. The editor then sets the link and the snapshot on the vector of that cause. |

Both items bring the editor of the shown model to the front and stage the change there. The change reaches the file when you save the model, and Discard drops it. The row leaves the section after the save.

The picker lists only causes that can take the link. It leaves out a cause that already links a Tetra Vector, and a cause that names its vector by reference. A cause holds one Tetra Vector link, so a second vector needs its own cause.

If the editor shows no drawer and no change, run the menu item again.

## Likelihood and suggested controls from rules

A vector rule can give every Tetra Vector it emits a likelihood and a list of suggested controls. The rules live in a `*.vector-rules.yaml` file that the Tetra manifest declares under `vectorRules`.

```yaml
# example-ot.tetra.yaml
vectorRules:
  - vendor-access.vector-rules.yaml
```

```yaml
# vendor-access.vector-rules.yaml
rules:
  - ruleId: vendor-vpn
    title: Vendor VPN into the control zone
    program:
      - 'model |> where("item.id == \"ch-vendor-vpn\"") |> emit("vector")'
    likelihood: Likely
    suggestedControls:
      - tpl-control-vendor-mfa
```

| Field | Content |
|---|---|
| `likelihood` | A label of the likelihood scale in the model's risk matrix, such as `Likely`. A key of the scale also matches. |
| `suggestedControls` | Control template ids. |

Both fields are optional. Tetra does not read the risk matrix, so it passes the label on unchanged. Lamina converts the label to a position on the scale of the model.

Several rules can emit the same channel or flow. The vector takes the likelihood of the first rule, in manifest order and then file order, that names one. When two rules name different likelihoods, Tetra reports a warning with both rule ids. The vector suggests the controls of every emitting rule, without duplicates.

The snapshot holds the likelihood and the controls, so a change to either marks each linked vector as Changed. A snapshot taken before a rule set them stays Current.

Lamina checks the values of each linked vector when a manifest opens and when a YAML file under its folder is saved. It writes a Problems-panel warning in two cases.

| Warning | Cause |
|---|---|
| The Tetra Vector has a likelihood that is not a label of this model's likelihood scale. | The label matches no label or key of the risk matrix. Match the spelling, or declare the scale. |
| The Tetra Vector suggests a control that is not a control template of this model. | The id names no control template in the templates the manifest declares. |

The check skips a vector that is Gone, which has its own warning. A model whose linked vectors carry no likelihood and no controls gets no warning.

### Replace an unresolved value

Each of the two warnings has a Quick Fix. Open the Quick Fix menu on the warning in the Problems panel, or on the manifest line it marks, and choose **Replace likelihood** or **Replace control**. Lamina opens the manifest in the diagram editor and shows a dialog.

The dialog names the value that the model cannot resolve and the rules that emit it. For a likelihood, the list holds the labels of the likelihood scale of the model, without Not Credible. For a control, the list holds the control templates of the model. Choose a value and select **Replace**.

Lamina writes the value into the `*.vector-rules.yaml` file of each rule that emitted the vector. The comments and the other rules in the file stay as they are. The cause and its snapshot do not change.

A rule can emit many Tetra Vectors, so the new value applies to all of them. Each linked vector that the rule emits shows as Changed until you update its snapshot. The warning clears when the rule file is saved.

The edit needs a licence that allows editing. A rule file that lies outside the folder of the Tetra model is not changed, and the dialog shows the reason.

## Resolve a Changed or Gone vector

A cause whose linked Tetra Vector is Changed or Gone shows a banner in its props drawer, above the suggested controls. The banner names the key of the Tetra Vector and offers the actions below. Each action stages a change to the vector of the cause, and nothing is written until the user selects Save all.

| State | The banner shows | Action | What the action writes |
|---|---|---|---|
| Changed | Each changed field with its old and new value. | Update snapshot | Replaces the snapshot with the live snapshot and sets the link name and the vector label to the live label. Sets the base likelihood to the converted likelihood, and the suggested controls to the live control ids that are control templates of the model. |
| Gone | The reason Tetra no longer derives the key. | Remove link | Deletes the link and the snapshot. Changes no other field. |
| Gone | The same reason. | Keep as hand-written | Deletes the link and the snapshot, then sets the label, base likelihood and suggested controls from the stored snapshot by the same rules as Update snapshot. |

Update snapshot leaves the base likelihood as it was when the live vector names no likelihood. It does the same when the label matches nothing on the scale. The suggested controls follow the same rule for control ids.

When an action changes the base likelihood, the banner shows the value before and after as a label of the likelihood scale. The inherent risk of the cause does not read the base likelihood, so no action changes the risk of the cause.

The banner reads the saved model. It hides as soon as the staged change resolves the link, and it checks the model again after a save or reload. When editing is not allowed, the buttons are disabled.
