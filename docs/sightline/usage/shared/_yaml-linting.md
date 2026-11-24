When a model loads, Tetra, Lamina and Metron list their findings in the Problems panel. Each finding names the source it came from.

## Open a Finding

1. Open the **Problems** panel.
2. Select a finding.

The app opens the source of the finding at the relevant line.

## YAML Finding Locations

- A finding about one entry, such as a device or a requirement, opens at the first line of that entry. It does not open at a field inside the entry.
- A finding about a top-level setting of the model opens at the line of that setting.
- A finding that names no entry and no setting opens its source at the top.
- A Metron finding about an assessment or a findings record opens that record, also when the assessment sits in a subfolder of the workspace.
- A Metron warning about the packages that a requirement package uses opens the requirement package.

## A Listed File That Is Missing

The manifest of a Tetra model or a Metron requirement package lists the model's files under `files:`. When a listed file is missing or cannot be read, the model loads from the files that exist. The Problems panel lists an error on the `files:` entry in the manifest. The error names the file and the reason it could not be read.

Entities that the missing file defines are absent from the model. A reference to one of them shows its own finding, such as an unknown member.

To clear the error, add the file or remove its entry from the manifest.

## Findings Outside the Open Model

In the web shell, the file viewer shows the files of the open model. A Metron finding in a file outside that model opens that file at its line. The file list marks the file as outside the model.

A finding in a file that the Metron workspace does not hold opens the requirement package. The finding says that the file is not one of the model's files.

## A Model That Does Not Load

A model file that the parser cannot read stops the model from loading. The editor shows the error. The Problems panel lists one error that names the file and the line where the parser stopped.

Lamina and Tetra read each model file as written, and then expand its templates. The reported line is the line in the file.

A reference such as `{{> const.example}}` that names no known template field or constant also stops the model from loading. The error names the file, the entry and field that hold the reference, and the reference.

A `{{> template}}` reference takes its text from the template that its entry names in `template:`. An entry that uses one without a `template:` stops the model from loading, and the error names the file, the entry and the field.
