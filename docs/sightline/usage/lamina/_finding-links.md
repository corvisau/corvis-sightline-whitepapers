A rule in a Tetra rule package creates a record for a channel or flow of a Tetra model. A cause in Lamina links that record from its vector and pins what it last accepted on the link. Lamina then reports when the record changes.

The props drawer of a cause has an **Open in Tetra** button for each link. See [Jump between a cause and its Tetra channel or flow](/docs/sightline/usage/shared/finding-jump).

## What a linked vector holds

A vector that links a record carries a record link in `links`. After the first Accept it also carries a `snapshot`.

| Field | Content |
|---|---|
| `links` | One entry with `scope: record` for each linked record. The `id` is the id of the record. The `instance` is the id of the Tetra model. The `name` is the title of the record when the link was made. A link to a ticket or a URL can sit in the same list. |
| `accepted` | A member of each record link. It holds a hash of the data, a hash of the links and the data that the record had when the cause last accepted it. |
| `snapshot` | The values that the linked records supplied at the last Accept. The proposable values are `label`, `description` and `tags`. |

```yaml
causes:
  - id: C001
    label: Remote access to the control zone
    vector:
      id: V001
      label: Example vendor VPN
      links:
        - scope: record
          instance: example-ot-model
          id: example-rules/cross-zone-relation:example-ot-model/relationId=ch-vendor-vpn,zoneId=z-control
          linkType: vector
          policy: manual
          name: "Relation enters a zone: z-control, ch-vendor-vpn"
          accepted:
            dataHash: 1a2b3c4d5e6f7
            sourceHash: 7f6e5d4c3b2a1
            data:
              label: Vendor VPN enters the control zone
      snapshot:
        label: Vendor VPN enters the control zone
```

The data is the answer that the rule computed. The links of the record are the versions of the channels, flows and zones that the record depends on. The two hashes are separate, so Lamina can tell a new answer from a changed dependency. The policy is always `manual`: the pin moves only when a person accepts.

## The states

Lamina compares the pin on the link with the record now.

| State | Meaning | Action |
|---|---|---|
| Current | Both hashes match the pin. | None. |
| Data changed | The rule computed different data. | Accept |
| Source changed | An entity that the record depends on changed, and the data did not. | Accept |
| Data and source changed | Both changed. | Accept |
| Not accepted yet | The link holds no pin. | Accept |
| Closed | The rule no longer creates the record. | Remove link |
| Not found | No record answers to the id. The block shows the name that the link holds. | Remove link |
| Cannot be read now | The register of the record cannot be read. | None. Open the block again once the register is readable. |

When several states apply, the block shows the first in this order: Cannot be read now, Not found, Closed, Not accepted yet, then the change states. A change that is reverted returns the link to Current.

## Work with the links in the props drawer

The props drawer of a cause shows a **Records** section with one block for each link. A changed link lists what moved, for example `type 'system' -> 'security'`. The list comes from the data that the pin holds, or from the journal of the record when the pin holds none. Without either, the block shows the state alone.

- **Accept** pins the link at the current hashes and data. It also copies the label, description and tags that the record supplies into the snapshot of the vector. It acts on one link and leaves the pins of the other links alone.
- **Withdraw approval** removes the snapshot and keeps every pin. The section shows the approved values and this button while a snapshot exists.
- **Remove link** deletes the link. It changes no other field.
- **Open in Tetra** opens the Tetra model at the first channel or flow that the record depends on.

Accept, Withdraw approval and Remove link each stage one change to the vector of the cause. Nothing is written until the user selects Save all. The block returns to Current as soon as the change is staged. When editing is not allowed, the buttons are disabled.

The links list in the generic **Links** field of the vector shows no row for a record link. Editing a ticket link in that field leaves every record link and its pin as they were.

## Choose the scope of the inbox

Two manifest fields set the scope of the records that the Findings dialog lists.

| Field | Content |
|---|---|
| `tetraZones` | The ids of the Tetra zones in scope. |
| `tetraModel` | The id of the one Tetra model to list. Without it, the dialog lists the records of every Tetra model that the workspace holds. |

```yaml
files:
  - causes.yaml
tetraModel: example-ot-model
tetraZones:
  - z-control
  - z-vendor
```

The Model Properties drawer edits both fields, so the manifest needs no hand edit. Open it in one of two ways:

1. Select **Model** in the toolbar.
2. Right-click an empty part of the canvas or of the Tabular Editor, then choose **Edit…**.

Choose a Tetra model, then select the zones in scope. The zone list shows the zones that the chosen model defines. While no model is chosen, it shows the zones of every Tetra model in the workspace, in any folder.

Save the model to write `tetraModel` and `tetraZones` to the manifest. A declared zone that the chosen model does not define stays selected, with the note "not defined in this Tetra model". Clear its checkbox to remove it. Choose **(any model)** and clear every zone to remove both fields from the manifest.

## List the records that no cause links

The **Findings** button in the toolbar opens the Findings dialog. Below the findings of Metron, a **Tetra records** list shows the records in scope that no cause of the open model links. Records from a rule with `hide` set do not appear, and a closed record does not appear.

Select **Create cause** on a row to open the new cause drawer. The drawer links the record, pins its current state and approves it into the snapshot. The row leaves the list after the save.

| Message | Cause |
|---|---|
| Declare tetraZones in the model manifest to list Tetra records. | The manifest has no `tetraZones`, or the list is empty. |
| Tetra model example-ot-model is not in this workspace. | `tetraModel` names a Tetra model that the workspace does not list. |
| Every Tetra record in scope has a cause. | Every record in scope has a link. |

## Problems panel

The Problems panel shows a warning for a link in one of five states. These are Data changed, Source changed, Data and source changed, Closed and Not found. The changed warning names the cause, the record and what moved. A link that reads Cannot be read now appears as an error, because Lamina cannot check it. A current link and a link that is not accepted yet give no entry. A deployment that refuses the read gives no entry either.

## Which manifests these reads cover

The Problems-panel warnings, the Model Properties choices and the **Records** section all find a model by its `meta.id`. They read a manifest that meets both conditions:

- The manifest sits in the first workspace folder and declares a `meta.id`.
- No other `*.bowtie.yaml` manifest in the workspace declares the same `meta.id`.

For any other manifest, the Problems panel shows no record warnings. The Model Properties drawer shows "Could not list Tetra models" in place of the choices.

## Models from before rule packages

A link with target kind `entity` that sits beside a `tetraSnapshot` field links a cause to a Tetra Vector. Lamina keeps both in the file and does not read them. The props drawer shows no block for them, and the Problems panel reports nothing for them.

Remove the link and link a record instead.

## Models from before record links

A record link with target kind `finding`, and accepted hashes kept in an `acknowledged` field, are errors when the model loads. A migration tool converts a model in the earlier format. Ask Corvis for the tool that matches your release.

The migration writes each link as a record link and moves its accepted hashes onto the link. The hashes carry no data, so the block lists what moved from the journal. A link that names no Tetra model is reported and left for a hand edit. A migrated link reads as changed until a person selects Accept.
