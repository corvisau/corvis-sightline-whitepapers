Tetra generates an entity inventory as an `.xlsx` or `.docx` file from the `reports` entries of a manifest. Choose **Export ▾** in the toolbar, then **Reports…**.

Lamina and Metron use the same report generator pipeline. Tetra adds two built-in generators, one for Excel workbooks and one for Word documents.

## Manifest authoring

Add one entry per report under the manifest's `reports:` array.

```yaml
reports:
  - id: inventory-xlsx
    name: Entity Inventory
    format: xlsx
```

| Field | Required | What it does |
|---|---|---|
| `id` | yes | Unique within the manifest. Identifies the report in the Reports picker. |
| `name` | yes | Human-readable label shown in the picker. Also the source of the output filename when `template` is absent. Each run of non-alphanumeric characters becomes `-`. |
| `format` | yes | `xlsx` or `docx`. Selects the Excel generator or the Word generator. |
| `dataSlice` | no | A `manifest.dataSlices[].id` to filter the model before the inventory is collected. Absent means the full model. |
| `template` | no | Path, relative to the manifest file, to a user-supplied template. An `xlsx` report without one gets a fresh multi-sheet workbook. A `docx` report requires one. |

A `docx` report with no `template` fails before anything is written.

## What the inventory contains

Every report collects the same flat entity inventory, with one collection per block-diagram entity type: `zones`, `systems`, `groups`, `devices`, `networks`, `channels`, `information` and `flows`.

A device row also carries its resolved `connections` ids, which are the ids under the device's `connections:` field.

A channel row carries `nodeA` and `nodeB`.

The inventory lists only the entities you authored.

## Excel report

The Excel report renders the inventory to an `.xlsx` workbook. Without a `template`, it writes a fresh multi-sheet workbook. With a `template`, it stamps the template.

Select the Excel generator with the `xlsx` format:

```yaml
reports:
  - id: inventory-xlsx
    name: Entity Inventory
    format: xlsx
```

### Excel templates

The report stamps a template sheet when both conditions hold:

- The sheet name matches one of the eight collections above, ignoring case.
- The sheet contains a `{{#rows}}` and `{{/rows}}` block-marker pair.

The report leaves any other sheet, such as a cover page, untouched. Columns inside the block use token placeholders such as `{{ id }}` and `{{ label }}`.

## Word report

The Word report fills a manifest-supplied `.docx` template with the inventory. Tetra ships no template of its own, so every `docx` report needs a `template` on its manifest entry.

```yaml
reports:
  - id: inventory-docx
    name: Entity Inventory
    format: docx
    template: reports/inventory.docx
```

### Word templates

Placeholders use `{...}` delimiters. A table body repeats with `FOR` and `END-FOR` over an inventory collection.

```
{FOR d IN devices}{$d.id} — {$d.label}{END-FOR d}
```

A shorter form prints only the id of each device:

```
{FOR d IN devices}{$d.id} {END-FOR d}
```

## Errors

A failed report shows an error message and writes no partial file. The message names the cause:

- An unknown report id
- An unresolved `dataSlice`
- A missing or absent template
