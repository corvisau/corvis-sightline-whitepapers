The Sightline API skill includes a Metron assessment helper. An agent records outcomes, evidence, notes and status on the pairs of an assessment by requirement label and entity name, and saves them together in one version-checked request. The helper finds each pair's ids and versions and the package's scheme itself.

You need Node.js 22 or later, the complete skill folder, and an approved Metron storage API endpoint. Your operator supplies the skill folder and the endpoint. Set `SKILL_DIR` to that folder. The helper reads its connection from `.sightline.env` in the working directory, as described in [Build models with a local agent](/docs/sightline/usage/tetra/agent-authoring#supply-the-connection-once).

Put your organisation's modelling conventions at the root of the working directory, in `SIGHTLINE.md` and `SIGHTLINE_METRON.md`. The agent reads them before each task on a Metron assessment and follows them, with `SIGHTLINE_TETRA.md` when it reads the assessed Tetra model. When `SIGHTLINE.md` names a security standard, such as `SIGHTLINE_SECURITY.md`, the agent reads it for an assessment against that standard, and for a task that writes the requirements and rules that test it.

## Read the Assessed Model

`model <assessment-id>` lists the Tetra model of an assessment as a requirement's scope rule reads it. Each item shows its kind, name, zone type, target security level, tags and the zone that holds it, and each channel and flow shows the zones of its two ends. `--kind` narrows the list to one kind. Use it to write a scope rule, and to count the items a requirement should cover.

## Check the Rules of an Assessment

`requirements <assessment-id>` lists each requirement with its scope rule and its assessment rule, and the rule package that holds each one. It lists under `unresolved` each rule that no package that the requirement package uses holds, and each used package that the assessment cannot find. Such a requirement has no pairs, or no outcomes.

`lint <rule-file>` checks a rule file, or a package's rules file, for the expression forms that do not work as they read, such as a test that mixes `&&` and `||` inside one filter. It needs no server. It reads the text only, so a rule that it passes can still give the wrong outcomes.

## Open an Assessment

```bash
node "$SKILL_DIR/scripts/metron.mjs" assessments
node "$SKILL_DIR/scripts/metron.mjs" open A-EXAMPLE1
node "$SKILL_DIR/scripts/metron.mjs" pairs --requirement OWN-1
```

`assessments` lists the assessments with their package and model. `open` reads one into a local draft and lists its requirements, the number of pairs of each, and the outcome and status names the package's scheme accepts. `pairs` lists pairs with their outcome, status, evidence and notes; `--requirement`, `--entity`, `--outcome` and `--status` narrow the list, and `--orphans` keeps the orphans: pairs whose entity the last run found out of scope. The default draft is `.sightline-drafts/metron.json`.

A pair exists for each entity that a requirement's scope selects. An assessment that has not been run has no pairs yet.

## Stage Results

```bash
node "$SKILL_DIR/scripts/metron.mjs" assess OWN-1 "Office,Plant" pass --evidence "Owner list v2"
node "$SKILL_DIR/scripts/metron.mjs" assess FW-1 all pass --except "Field RTU" --evidence "Firewall export, May"
node "$SKILL_DIR/scripts/metron.mjs" status OWN-1 all done
```

`assess` sets the outcome of each named pair. Name the entities by label or id, separated by commas, or write `all` and leave some out with `--except`. `--evidence` adds a note as evidence and can be repeated, `--url` adds a link, and `--notes` sets the notes. Evidence and notes that a command does not give keep their current values. An outcome or status can be given by its id or its label.

`attribute` records the state of one attribute on each named pair: `met`, `partial` or `not-met`, with an optional `--note`. Name the attribute as its attribute set does, such as `attribute DEV-1 "Filter PLC" "Default accounts removed" met`. `pairs` lists each pair's attributes and their states. Recording an outcome or an attribute keeps the pair's other attribute results and its links to general evidence.

`status` moves pairs to a status of the scheme, such as `in-progress` or `done`. A pair needs an outcome before it can be done, and a done pair goes back to `in-progress` before its outcome changes; both can be staged in the same session. `staged` lists the staged commands, `undo` drops the last one and `discard` drops them all.

## Move an Orphan's Results

```bash
node "$SKILL_DIR/scripts/metron.mjs" pairs --orphans
node "$SKILL_DIR/scripts/metron.mjs" move DEV-1 old-plc "Filter PLC"
```

An orphan keeps its results after its entity leaves scope, for example when the entity is replaced by a new item with a new id. `move` copies an orphan's outcome, evidence, notes, links to general evidence and attribute results to the in-scope pair that replaces it. The orphan's status comes too, except that a done orphan arrives as in-progress, so that a person reviews it for the new entity. `move` refuses a done target, and a target that already holds results unless `--replace` is given. `--archive` also archives the orphan in the same save, which deletes it and its results. Run the assessment first: a pair becomes an orphan, and a new entity gets its pair, when the assessment runs.

## Save

```bash
node "$SKILL_DIR/scripts/metron.mjs" save "Record the May audit"
```

`save` sends every staged change in one request against the assessment version that was opened. It then reads the assessment again and reports `"ok":true` when every staged outcome and status is recorded. When the assessment changed on the server in the meantime, the save is refused and nothing is written; open the assessment again and restage the results.

The server records who assessed a pair and when. Package authoring, rule runs, shared assessments and findings remain available through the skill's file-level and API workflows.
