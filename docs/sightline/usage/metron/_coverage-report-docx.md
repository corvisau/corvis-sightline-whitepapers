The coverage report writes the coverage view of a Metron package into a Word template that you supply.

## What the report contains

| Section | Rows |
|---|---|
| `coverageByPractice` | One row for each flagged gap, grouped across every finding that flagged it |

The columns match the **Coverage** panel of Metron.

The report never shows attribute satisfaction. Satisfaction is a live working signal and not a human assertion. A reader of a formal record could take satisfied to mean resolved.

The report does not filter by status. Finding status appears as a column.

## The template is yours

The report ships no template. Declare a template on the `reports[]` entry of the manifest, as for the Tetra Word reports:

```yaml
reportScriptsDir: my-generators   # optional, for custom bundles
reports:
  - id: coverage
    name: Coverage report
    generator: metron-docx
    template: reports/coverage.docx
    output: outputs/coverage.docx
```

Placeholders in the template use `{...}` delimiters. A table body repeats with `FOR` and `END-FOR`:

```
{FOR row IN coverageByPractice}{$row.category}  {$row.practice}  {$row.attribute}  {$row.findingStatus}  {$row.findings}{END-FOR}
```

The template can also use these top-level values: `generatedDate`, `reportName`, `packageTitle` and `packageVersion`.

## Run the report

Run **Metron: Export Coverage Report** for a package, or **Metron: Export Assessment Coverage Report** for an assessment. Both commands offer the reports that the manifest declares.

The report works in package mode (`*.metron.yaml`) and in assessment mode (`*.assessment.yaml`). An assessment that declares its own `reports[]` renders through the same generator without change.
