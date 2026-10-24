External data is a set of reference tables that Metron loads next to an assessment, such as an audit export or a control-attestation feed. Metron shows the tables beside the practice you assess. A rule can join against them to suggest an outcome.

The tables come from systems that know nothing about Metron requirements or Tetra entities. Metron asks almost nothing of them and keeps every column the source produces. Metron reads these tables and never changes them. The source system maintains them.

---

## Find the tables

The **External data** tab appears in an assessment only when the assessment has at least one table. If the tab is missing, no table applies to the model of this assessment. Your administrator loads tables into the workspace.

The tab shows one grid for each table. The heading of a grid shows the table label and its row count. Each grid has a search box. The grid shows every column that the rows carry, in the order the source gives them.

## Show and hide columns

1. Open the **External data** tab.
2. Select **Edit columns** above the grid of the table.
3. Tick the columns to show, and untick the columns to hide.
4. Drag a column handle, or use the keyboard, to change the order.
5. Drag the edge of a column header to change its width.

Each table keeps its own choice, so a change in one table does not change another table. Metron keeps the choice on your computer.

## Match rows to a practice

Some tables name the requirement and the entity that each row concerns. Metron matches those rows to the pairs on the worklist. When you open the **Assess** drawer of a pair, the drawer lists the matching rows below the pair, grouped by table.

A row that names a zone matches that zone only. It does not match the entities inside the zone. A rule can roll rows up across a zone, as the next section shows.

A table that does not name a requirement or an entity never appears in the **Assess** drawer. A rule reaches it through its own conditions. This is the usual case for a feed that a machine generates. Both kinds of table show in full on the **External data** tab.

---

## Use a table from a rule

A rule that assesses a requirement can read any table that is visible to the assessment. In the program, the identifier of the table is the name of a register. See [Rules](/docs/sightline/usage/metron/rules) for how rules are written.

### End the program on model entities

A rule returns one row for each entity it assesses. Metron records the outcome against the entity whose id the row carries. A program must therefore end on model entities.

This program is correct. It filters the rows, then selects the model entities that the rows name.

```
attestations |> where("item.status == \"healthy\"") |> save("ok")
model |> where("ok |> anyOf(.hostname == item.id)") |> set("outcome", "'compliant'") |> save(result)
```

This program is wrong. It returns row ids, and row ids are not entities, so nothing is assessed.

```
attestations |> where("item.status == \"healthy\"") |> set("outcome", "'compliant'") |> save(result)
```

The wrong program is valid, but it assesses nothing. Metron raises a warning that names the ids it could not match. The warning reads:

> rule '…' returned N id(s) that are not model entities

If you see this warning, the program has this mistake.

### Join rows to entities

Inside `anyOf(...)` and `noneOf(...)`, `.field` is the field of the row and `item` is the entity under test.

```
model |> where("ok |> anyOf(.hostname == item.id)")
model |> where("ok |> anyOf(.zoneId == item.zoneId)")
model |> where("bad |> noneOf(.hostname == item.id)")
```

The first line matches a row to an entity by id. The second line matches any row in the zone of the entity. The third line selects the entities that have no row in `bad`.

The second form is the zone roll-up. The rule engine derives `zoneId` onto every entity. One condition therefore tests whether any attested control in the zone of the entity is healthy.

Follow these syntax rules:

- Combine conditions with `&&`. The word `and` is not accepted.
- Do not write arrow functions such as `r => r.x`. Use the `.field` form. Alternatively, filter into a register with `where` first, because `where` takes a full condition.

### The rule outcome is a draft

A rule-suggested outcome shows on a draft pair, so a person owns the completion. A rule never moves a pair to a `done` status. The assessor reviews the suggested outcome and completes the pair.

---

## What external data does not do

- It is not assessment state. Metron does not save a row onto a pair, and it matches rows again on every read.
- It does not trigger revalidation. A completed pair is not flagged when its external rows change. Re-open the pair to assess it again.
- It does not reach generated documents. Tetra and Lamina do not use it.
