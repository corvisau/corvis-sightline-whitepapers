Arbitrary reference tables loaded alongside an assessment, such as an audit export or a
control-attestation feed. Metron surfaces them next to the practice you are assessing,
and lets a rule join against them to suggest an outcome or, opt-in, complete it.

The governing idea is that these files are dumb. They come out of systems
that know nothing about Metron requirements or Tetra entities. Metron therefore asks
almost nothing of them: a table needs an `id`, and nothing else is required. Metron
carries every column your source produces through untouched.

---

## Where they live

```
<workspace>/external/**/*.external.yaml
```

Workspace-rooted and glob-scanned, the same way authored findings are.
Sharding across several files is fine, and the files merge.

External data is read-only in Metron. It is maintained at its source, so there
are no editing affordances, and Metron never writes back to these files.

## The file

```yaml
tables:
  - id: attestations              # required; also the name a rule pipes from
    label: Control attestations   # optional; shown in the UI, defaults to id
    modelId: grid-primary         # optional; see "Narrowing to one model"
    rows:
      - hostname: hmi01.example
        zoneId: Z-SCADA
        status: healthy
      - hostname: plc09.example
        zoneId: Z-SCADA
        status: degraded
```

Three row keys are **reserved**: `id`, `practiceRef` and `scope`. Every other key
is arbitrary and passes through verbatim, never validated, renamed or dropped.

All three reserved keys are optional.

## Two table shapes, both first-class

### 1. Practice-shaped — opts in to the built-in join

A practice-shaped table carries `practiceRef` and `scope`, so Metron itself can match rows to pairs and
show them in the worklist's Assess drawer. This requires the file to know
your requirement ids.

```yaml
tables:
  - id: audit-2026
    label: 2026 Internal Audit
    rows:
      - id: AF-001
        practiceRef: ACM-1        # a Requirement.id
        scope: DEV-HMI-01         # an exact entity id, or '*'
        severity: high
        summary: Shared operator account on the SCADA PLC
```

- `practiceRef` must equal the requirement's `id`.
- `scope` is an exact entity id, or `'*'` meaning every in-scope entity of
  that practice.
- There is no container descent. A `scope` naming a zone matches that zone
  entity and nothing inside it. A rule rolls rows up across a zone; see below.

A row missing either key never matches a pair. That is not an error.

### 2. Dumb — no Metron knowledge at all

A dumb table carries neither key. It never appears in the Assess drawer, and is reached
purely by a rule's own predicates. This is the normal shape for a
machine-generated feed.

Both shapes render in full on the **External data** tab, which derives its
columns from whatever keys the rows actually carry.

## Column visibility

Each rendered table has its own **`[Edit columns]`** picker. The picker lists that table's derived columns:
reserved keys (`id`/`practiceRef`/`scope`, when present) first, then every
other key in first-seen row order. Each table's picker is independent of every other table's.
The column set comes from the data rather than a fixed schema. Metron therefore
keeps each table's choice apart, by that table's own `id`. A resized column's width is kept the same way.

## Row ids

`id` is optional in the file. Metron gives any row without one the id
`<tableId>::<index>` at load time. Author an id only to give a row a stable handle.

## Reserved table ids

A table may not be called `model` or `result`. A table's `id` becomes a
register name inside every rule program. `model` would shadow the model itself,
and `result` is the reserved target every `save`/`add`/`remove` writes to. A
table using either name is skipped with a diagnostic.

Metron also reports a diagnostic and skips a duplicate table id or a file that
fails to parse, then carries on loading.

## Narrowing to one model

`modelId` is optional and narrowing:

- **declared**: the table is visible only to an assessment bound to that model.
- **absent**: the table is workspace-wide reference data, visible everywhere.

`modelId` gives no correctness guarantee. Where matching the right thing matters,
match on the row's own columns.

---

## Using a table from a rule

A `rule`-method requirement's Bonsai program can pipe from any visible table by
its `id`. For rules in general, see the [Rules](/docs/sightline/usage/metron/rules) page.

### Get this right first

`emit` tags whatever set is piped into it. Metron records an outcome for a
pair by matching the emitted id against the model's entities, so a program must
end by emitting model entities.

```yaml
# CORRECT - filter the rows, then select the MODEL entities they name.
program:
  - attestations |> where("item.status == \"healthy\"") |> save("ok")
  - model |> where("ok |> anyOf(.hostname == item.id)") |> emit("compliant")
```

```yaml
# WRONG - this emits ROW ids, which are not entities. Nothing is assessed.
program:
  - attestations |> where("item.status == \"healthy\"") |> emit("compliant")
```

The wrong version is syntactically valid and assesses nothing, so Metron raises a
warning naming the ids it could not match. If you see

> rule '…' emitted N id(s) that are not model entities

that is this mistake.

### Joining

Inside `anyOf(...)` / `noneOf(...)`, `.field` is the row's field and `item`
is the entity being tested.

```
model |> where("ok |> anyOf(.hostname == item.id)")        # row -> entity by id
model |> where("ok |> anyOf(.zoneId == item.zoneId)")      # any row in my zone
model |> where("bad |> noneOf(.hostname == item.id)")      # no bad row for me
```

The second form is the zone roll-up. The rule engine derives `zoneId` onto every
entity, so one predicate tests whether any attested control in the entity's zone is healthy.

Two syntax notes:

- Combine conditions with `&&`. The word `and` is not accepted.
- The syntax has no arrow-function lambdas (`r => r.x`). Use the `.field` form,
  or filter into a register with `where` first. `where` takes a full predicate,
  so the two-statement form avoids the limitation.

### Letting a rule complete the pair

By default a rule-suggested outcome lands as a draft, so a human owns the
completion boundary. A requirement can opt out:

```yaml
assessment:
  method: rule
  ruleRef: attestation-join-rule
  autoComplete: true
```

With `autoComplete: true`, a recognised emitted outcome also moves the pair to the
first `done`-fundamental status your scheme declares. Declaration order
decides, and a scheme with no custom statuses gets the built-in `done`.

The action is recorded as automated: `assessedBy` becomes
`rule:<ruleId>`, and the `complete` log entry's `cause` names the rule that
signed it off. An unrecognised emitted outcome never auto-completes; that
sentinel waits for a person.

`autoComplete` is valid only with `method: rule`, and a manual requirement rejects it.

---

## What external data is not

- **It is never assessment state.** A row is never persisted onto a pair the way
  linked evidence is. Matching is recomputed on every read.
- **It is never a drift comparand.** A completed pair whose external rows later
  change is not flagged for revalidation. `requiresRevalidation` keeps its
  existing meaning (requirement and entity versions). Re-open the pair if you
  want it reassessed.
- **It does not reach generated documents**, and it is not exposed to Tetra or
  Lamina.
