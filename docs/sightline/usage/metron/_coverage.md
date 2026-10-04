The **Coverage** tab is a read-only view over findings, grouped by flagged gap.
It never feeds back into compliance scoring.

## By practice

One row per flagged gap, grouped by category:

| Column | Meaning |
|---|---|
| Category | The practice's category path. |
| Practice | `Title (ID)`. |
| Entity | The assessed entity. |
| Attribute | The attribute, or `(whole practice)`. |
| Finding status | The status of every finding that flagged this gap. |
| Findings | Those findings' ids. |

A gap flagged by two different findings is one row listing both.

## Nothing is filtered out for you

The view does not hide a gap because its finding is `resolved`. Finding status
is a filterable column. Use the grid's **Filter** control to narrow to open
work.

A reference whose requirement or attribute set no longer exists still produces
a row. The practice shows as its raw id with category `(unknown)`. An
unresolvable attribute shows as its raw name.

## Attribute met

The view carries an **Attribute met** column showing whether that gap's
attribute is met on the live pair currently:

| Value | Meaning |
|---|---|
| `Yes` | The attribute is currently **Met** in the Assess drawer |
| `No` | It is not: **Partial**, **Not met**, or never assessed |
| *(blank)* | Nothing to say: the gap covers the whole practice, so there is no single attribute to meet |

This is the one column on the row that is live rather than pinned. Every
other value describes the gap as it was flagged (at that requirement and entity
version); this one describes the world as it is now. A `Yes` therefore does not
mean the finding is resolved, because the finding is a human assertion and only
a human revises it. It means someone has satisfied that attribute since, and
the finding may be worth revisiting.

It is deliberately absent from generated documents. A docx is the formal
record of what was asserted, and a reader could reasonably take "satisfied"
there as "resolved". See the Formal record vs live signal section in
[Assessment overlay model](/docs/sightline/usage/metron/assessment-overlay-model).

## Column visibility

The **`[Edit columns]`** picker lets you show, hide and reorder the view's
columns. A resized column's width persists the same way. Drag the handle at the left of a column header to move that column, or focus the handle and use Space and the arrow keys. The picker shows the same order.

## Generating a coverage document

See the [coverage report page](/docs/sightline/usage/metron/coverage-report-docx) for exporting this
view to a `.docx` template.
