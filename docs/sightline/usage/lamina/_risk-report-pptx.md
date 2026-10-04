The PowerPoint risk report mirrors the Word risk report, and it groups the risks by data slice. For every data slice in the manifest, the report writes these slides:

| Slide | Content |
|---|---|
| Divider | The name and the description of the data slice |
| **Events** | Event, causes, inherent and residual likelihood, outcomes, inherent and residual consequence, and highest risk. There is one sub-row for each outcome. The event, causes, likelihood and risk cells merge across the sub-rows. Each outcome keeps its own inherent and residual consequence. |
| **Preventative Controls** | Cause (with its threat vector), preventative controls, target effectiveness, current effectiveness and current effectiveness detail. There is one row for each control, without duplicates. The cause cell merges. |
| **Causes** | Name, inherent controls, preventative controls, inherent and residual likelihood, and events. The name shows the label, the full vector description and the id. |
| **Mitigative Controls** | Outcome, inherent controls, mitigative controls, target effectiveness, current effectiveness and current effectiveness detail. There is one row for each control, without duplicates. The outcome and inherent-controls cells merge. |
| **Outcomes** | Name, events, inherent controls, controls, and inherent and residual consequence for each category |

Each table continues onto as many slides as it needs. A row is never split across slides. When a group of merged rows crosses onto the next slide, the merged cell repeats there. The likelihood, consequence and risk cells carry the same colours as the Word report.

## The template is a style reference

The template deck holds five example slides, one for each table type, in report order: Events, Preventative Controls, Causes, Mitigative Controls and Outcomes. The report reads the header text, the column widths, the table style and the fonts of each example table. It reproduces them, then removes the example slides from the output.

In PowerPoint, you can change these in the template: column widths, header text, table style, theme accent, fonts and colours.

Do not add, remove or reorder columns. The report maps each column position to a fixed field. It stops with an error when the column count of an example table does not match.

## Prerequisites

The report needs the `python-pptx` package on the Python interpreter that the report uses. Install it with `pip install python-pptx`. Alternatively, point the `command` of the `reports[]` entry at an interpreter in a virtual environment.

## Run the report

Declare the report on the manifest:

```yaml
reports:
  - id: risk-pptx
    name: Risk Report (PowerPoint)
    generator: pptx
    template: reports/pptx-risk.template.pptx   # the style-reference deck, owned by the model
    output: outputs/pptx-risk.pptx
```

Then choose **Export**, **Reports…** in Lamina. Select the report. The generated slides use the **Title Only** layout of the template deck.

## Related

- [PNG export](/docs/sightline/usage/lamina/png-export)
- [Data slices](/docs/sightline/usage/lamina/data-slices)
