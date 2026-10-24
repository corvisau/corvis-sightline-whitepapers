Tetra generates an entity inventory as an Excel workbook or a Word document. Choose **Export ▾** in the toolbar, then **Reports…**. You cannot declare a report in the app yet.

Lamina and Metron use the same report generator pipeline. Tetra adds two built-in generators, one for Excel workbooks and one for Word documents.

## Run a report

1. Choose **Export ▾** in the toolbar.
2. Choose **Reports…**.
3. Select a report in the **Run Report** dialog.
4. Choose **Run**.

**Reports…** offers a report only when the model declares one. A report can filter the model through a data slice before it collects the inventory. Without a data slice, the report covers the full model.

In VS Code, a finished report lands in the model's `outputs/` folder and opens in its default application.

## What the inventory contains

Every report collects the same flat entity inventory, with one collection per block-diagram entity type: `zones`, `systems`, `groups`, `devices`, `networks`, `channels`, `information` and `flows`.

A device row also carries the ids of its resolved connections. A channel row carries `nodeA` and `nodeB`.

The inventory lists only the entities you authored.

## Template reference

This section is reference material for people who prepare their own report templates. It describes the placeholders that the generators accept.

### Excel report

The Excel report renders the inventory to an Excel workbook. Without a template, it writes a fresh multi-sheet workbook. With a template, it stamps the template.

#### Excel templates

The report stamps a template sheet when both conditions hold:

- The sheet name matches one of the eight collections above, ignoring case.
- The sheet contains a `{{#rows}}` and `{{/rows}}` block-marker pair.

The report leaves any other sheet, such as a cover page, untouched. Columns inside the block use token placeholders such as `{{ id }}` and `{{ label }}`.

### Word report

The Word report fills a template with the inventory. Tetra ships no template of its own, so every Word report needs one. A Word report with no template fails before anything is written.

#### Word templates

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
- An unresolved data slice
- A missing or absent template
