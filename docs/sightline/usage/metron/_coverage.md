The **Coverage** tab is a read-only view over findings, grouped by the pairs they
cover. It never feeds back into compliance scoring.

## By practice

One row per member pair of a finding, grouped by category:

| Column | Meaning |
|---|---|
| Category | The practice's category path. |
| Practice | `Title (ID)`. |
| Entity | The assessed entity. |
| Attribute | `(whole practice)`, because a finding covers a whole pair. |
| Finding status | The status of every finding that covers this pair. |
| Findings | Those findings' ids. |

A pair covered by two different findings is one row listing both.

## Nothing is filtered out for you

The view does not hide a pair because its finding is `resolved`. Finding status
is a filterable column. Use the grid's **Filter** control to narrow to open
work.

A member whose requirement no longer exists in the package still produces a row.
The practice shows as its raw id with category `(unknown)`.

## Attribute met

The **Attribute met** column shows whether one attribute is met on the live pair.
A finding covers a whole pair and names no single attribute, so the column is blank
on every row.

The column is absent from generated documents. A docx is the formal record of what
was asserted. See the Formal record vs live signal section in
[Assessment overlay model](/docs/sightline/usage/metron/assessment-overlay-model).

## Column visibility

The **`[Edit columns]`** picker lets you show, hide and reorder the view's
columns. A resized column's width persists the same way. Drag the handle at the left of a column header to move that column, or focus the handle and use Space and the arrow keys. The picker shows the same order.

## Generating a coverage document

See the [coverage report page](/docs/sightline/usage/metron/coverage-report-docx) for exporting this
view to a `.docx` template.
