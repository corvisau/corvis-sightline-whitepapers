When a model loads, Tetra, Lamina and Metron list their findings in the Problems panel. Each finding names the source it came from.

## Open a finding

1. Open the **Problems** panel.
2. Select a finding.

The app opens the source of the finding at the relevant line.

## Where a finding opens

- A finding about one entry, such as a device or a requirement, opens at the first line of that entry. It does not open at a field inside the entry.
- A finding about a top-level setting of the model opens at the line of that setting.
- A finding that names no entry and no setting opens its source at the top.
- A Metron finding about an assessment or a findings record opens that record, also when the assessment sits in a subfolder of the workspace.
- A Metron warning about the packages that a requirement package uses opens the requirement package.

## Findings outside the open model

In the web shell, the file viewer shows the files of the open model. A Metron finding in a file outside that model opens that file at its line. The file list marks the file as outside the model.

A finding in a file that the Metron workspace does not hold opens the requirement package. The finding says that the file is not one of the model's files.

## A model that does not load

A model file that the parser cannot read stops the model from loading. The editor shows the error. The Problems panel lists one error that names the file and the line where the parser stopped.

Lamina and Tetra expand templates before they parse a model file. For a file that uses templates, the reported line can differ from the line in the file. The error message says so.
