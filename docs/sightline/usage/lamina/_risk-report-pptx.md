The PowerPoint risk report mirrors the Word risk report, and it groups the risks by Risk. It needs a model that defines Risks, and it stops with an error for a model that defines none. Declaring a report entry is not available in the app yet. For every Risk in the model, the report writes these slides. A Risk with no path writes none, and a Risk reports the causes, events and outcomes on its paths and on the paths of the Risks it holds:

| Slide | Content |
|---|---|
| Divider | The name and the description of the Risk |
| **Events** | Event, causes, inherent and residual likelihood, outcomes, inherent and residual consequence, and highest risk. There is one sub-row for each outcome. The event, causes, likelihood and risk cells merge across the sub-rows. Each outcome keeps its own inherent and residual consequence. |
| **Preventative Controls** | Cause (with its threat vector), preventative controls, target effectiveness, current effectiveness and current effectiveness detail. There is one row for each control, without duplicates. The cause cell merges. |
| **Causes** | Name, inherent controls, preventative controls, inherent and residual likelihood, and events. The name shows the label, the full vector description and the id. |
| **Mitigative Controls** | Outcome, inherent controls, mitigative controls, target effectiveness, current effectiveness and current effectiveness detail. There is one row for each control, without duplicates. The outcome and inherent-controls cells merge. |
| **Outcomes** | Name, events, inherent controls, controls, conditional modifiers with their conditions, and inherent and residual consequence for each category. The conditional modifiers appear under **Conditional Modifiers** only for an outcome that has some. |

Each table continues onto as many slides as it needs. A row is never split across slides. When a group of merged rows crosses onto the next slide, the merged cell repeats there. The likelihood, consequence and risk cells carry the same colours as the Word report. The event likelihood and the highest risk follow the rating rule of the model. See [Risks](/docs/sightline/usage/lamina/risks).

## The template is a style reference

The template deck holds five example slides, one for each table type, in report order: Events, Preventative Controls, Causes, Mitigative Controls and Outcomes. The report reads the header text, the column widths, the table style and the fonts of each example table. It reproduces them, then removes the example slides from the output.

In PowerPoint, you can change these in the template: column widths, header text, table style, theme accent, fonts and colours.

Do not add, remove or reorder columns. The report maps each column position to a fixed field. It stops with an error when the column count of an example table does not match.

## Prerequisites

The report needs the `python-pptx` package on the Python interpreter that the report uses. Install it with `pip install python-pptx`.

## Run the report

Choose **Export**, **Reports…** in the toolbar. The **Run Report** dialog lists the reports that the model offers. Select the report, then select **Run**. On a model that defines Risks the dialog offers **Run for all Risks**, which runs the report once for each Risk. A report can name its Risk in the `risk` field of its entry in the manifest, and then it reports that Risk alone; without the field it reports the whole model.

The dialog lists only the reports that the model already offers, and it shows a hint when the model offers none.

In VS Code, the finished file lands in the model's `outputs/` folder and opens in its default application. The generated slides use the **Title Only** layout of the template deck.

## Related

- [PNG export](/docs/sightline/usage/lamina/png-export)
- [Risks](/docs/sightline/usage/lamina/risks)
