The Sightline API skill includes an incremental Lamina helper. An agent can add one cause, event, outcome or control at a time, link them, review a local draft and save the changes together. The helper writes the YAML, the files and the ids, checks the result against the schemas, and saves with a version check.

You need Node.js 22 or later, the complete skill folder, and an approved Lamina storage API endpoint. Your operator supplies the skill folder and the endpoint. Set `SKILL_DIR` to that folder. The helper reads its connection from `.sightline.env` in the working directory, as described in [Build models with a local agent](/docs/sightline/usage/tetra/agent-authoring#supply-the-connection-once).

Put your organisation's modelling conventions at the root of the working directory, in `SIGHTLINE.md` and `SIGHTLINE_LAMINA.md`. The agent reads them before each task on a Lamina model and follows them.

## Create or Open a Model

```bash
node "$SKILL_DIR/scripts/lamina.mjs" new "Example Pump Station Flooding"
node "$SKILL_DIR/scripts/lamina.mjs" open example-pump-station-flooding
```

`new` creates a dedicated directory and a starter model on the server, then opens its local draft. `open` reads an existing model. The default draft is `.sightline-drafts/lamina.json`. A Lamina draft and a Tetra draft are kept apart: each helper refuses the other's draft.

## Use Your Own Risk Matrix

A new model uses the built-in risk matrix. When your organisation rates on its own matrix, set it before any rating:

```bash
node "$SKILL_DIR/scripts/lamina.mjs" matrix set --likelihood "Rare,Unlikely,Possible,Likely,Almost Certain" --consequence "Insignificant,Minor,Moderate,Major,Severe" --categories "people,environment,financial=Asset damage" --levels "Low,Medium,High,Extreme" --grid "Almost Certain=M,H,H,E,E;Likely=M,M,H,H,E;Possible=L,M,M,H,E;Unlikely=L,L,M,M,H;Rare=L,L,L,M,H"
node "$SKILL_DIR/scripts/lamina.mjs" matrix
```

Each list runs from the lowest to the highest. `--categories` takes a key, or `key=Label` for a longer display label. Each `--grid` row names a likelihood and gives one level per consequence, from the lowest; a level can be written as its initials. `--probability` sets the value of each likelihood used when causes are combined; without it, a five-step scale takes the built-in values. A part you leave out keeps its current value.

The helper writes the matrix to its own file, `risk-matrix.yaml`, and points the model's manifest at it. `matrix` shows each scale with the number the model stores for each word, and the grid cell by cell. Likelihood and consequence words are then read through this matrix, and a category it does not declare is refused. `check` reports a rating that falls outside the scale or rates an undeclared category, which Lamina would otherwise drop when it loads the model. To move a rated model to a new matrix, set the matrix, then rate each item again in its words; `consequence O001 inherent safety=none` clears a rating in a category the new matrix no longer declares.

## Add Events, Outcomes and Causes

```bash
node "$SKILL_DIR/scripts/lamina.mjs" event "Pump station floods"
node "$SKILL_DIR/scripts/lamina.mjs" outcome "Electrical equipment damaged"
node "$SKILL_DIR/scripts/lamina.mjs" link E001 O001
node "$SKILL_DIR/scripts/lamina.mjs" cause "Main pump trips" --actor "Pump motor" --inherent Likely --current Unlikely --event E001
```

Each command prints the new item's id. The helper chooses the next free id, such as `C001`, `E001` or `O001`; `--id` chooses one instead. `link` joins a cause to an event, or an event to an outcome, and `unlink` removes the link.

A likelihood is given in the words of the model's risk matrix, such as `--inherent "Almost Certain"` with the matrix above. The built-in matrix uses `Very Unlikely`, `Unlikely`, `Possible`, `Likely` and `Very Likely`, and `Not Credible`. The helper stores the matching rating and refuses a word the matrix does not have, with the list of words it accepts.

## Add Controls and Ratings

```bash
node "$SKILL_DIR/scripts/lamina.mjs" control C001 "Standby pump" --category Technology --current Full --target Full --critical
node "$SKILL_DIR/scripts/lamina.mjs" inherent C001 "Overflow weir" --valid true
node "$SKILL_DIR/scripts/lamina.mjs" control O001 "Equipment raised above floor" --category Technology --current Full --target Full
node "$SKILL_DIR/scripts/lamina.mjs" consequence O001 inherent financial=4
node "$SKILL_DIR/scripts/lamina.mjs" consequence O001 residual financial=2
```

A control on a cause is preventative by default, and a control on an outcome is mitigative; `--type Detective` records a detective control. `--current` and `--target` take `None`, `Low`, `Partial`, `Full` or `Not Assessed`. `--current-detail` records the evidence behind the current effectiveness, and `--target-detail` the reasoning behind the target. `inherent` adds an inherent control, and `conditional <outcome> <label> --condition "<text>"` adds a conditional modifier. An inherent control or a conditional modifier has no effectiveness, so the helper refuses these options and zones on one. `consequence` sets inherent or residual ratings by category; a category the model's matrix does not declare is refused.

An outcome takes `--likelihood-reduction`, the number of levels it lowers the likelihood that reaches it on the matrix's likelihood scale. It also takes `--impact-features` and `--inherent-response` as free text, `--line-of-business` as a list, and `--material-risk` to flag it as a material risk.

`risk R001 "Flooding" --event E001` adds a Risk that holds the event with every cause that leads to it and every outcome it leads to, so it covers every path through the event. `--include` and `--exclude` add or leave out single causes, events and outcomes, `--risk` holds another Risk, `--all` selects every element, and `--folder` places the Risk in the risk tree. `--per-event-likelihood` sets how the Risk combines the causes of one event: `max`, `simple-damped-union` or `exposure-grouped`. Without it, the model's default applies. A model has only the Risks you add. `annotate C001 "<text>"` attaches a note to an item.

## Zones, Folders and Tags

```bash
node "$SKILL_DIR/scripts/lamina.mjs" cause "Remote compromise" --event E001 --folder "Site North" --tag remote,r1
node "$SKILL_DIR/scripts/lamina.mjs" tag C001 reviewed
node "$SKILL_DIR/scripts/lamina.mjs" untag C001 r1
```

`--zones` takes the ids of zones in the Tetra model. Outcomes and assessed controls take zones. A cause takes its zones from the Tetra flows it links. A control on an outcome without its own zones takes the outcome's zones, and a control on a cause has none. `--folder` places a cause, event or outcome in the element tree. A folder path joins names with `/`. A folder that differs from an existing one only by case is refused, so use the existing spelling. Causes, events and outcomes take `--tag`. A tag keeps its case, and a repeated tag is dropped. `tag` and `untag` add or remove some tags and keep the others.

## Use the Model's Templates

```bash
node "$SKILL_DIR/scripts/lamina.mjs" list template
node "$SKILL_DIR/scripts/lamina.mjs" cause --template tpl-cause-remote-access --event E001
node "$SKILL_DIR/scripts/lamina.mjs" control C001 --template tpl-control-endpoint-protection --current Low
```

`list template` lists the model's templates, with the kind of item each one is for. The commands that add a cause, event, outcome or control take `--template` with a template's id. The label is then optional. The item takes from the template what it does not give itself. Add a template for an inherent control with `inherent`, and one for a conditional modifier with `conditional`. The helper does not write templates; create and change them in Lamina.

## Inspect and Correct

`query` searches on any field and chooses what each result shows. `--missing <field>` keeps items without a field, and `--has <field>` keeps those with it. `--where` takes `field=value`, `field!=value`, `field~text`, or `>`, `<`, `>=` and `<=` with a number. `--show` picks the fields of each result, and `--show all` gives every field. For example, `query control --missing currentEffDetail --show label,owner` lists the controls with no evidence recorded. `fields` lists the fields that items hold, with a count and an example of each.

```bash
node "$SKILL_DIR/scripts/lamina.mjs" list cause
node "$SKILL_DIR/scripts/lamina.mjs" show C001
node "$SKILL_DIR/scripts/lamina.mjs" set C001 current Possible
node "$SKILL_DIR/scripts/lamina.mjs" rename C001 EX-C001
node "$SKILL_DIR/scripts/lamina.mjs" undo
```

`set` changes one field of an item, such as a label, a likelihood or a control's effectiveness. The helper's help lists the fields of each kind of item. `none` removes a list of tags or zones, a folder, a likelihood reduction or a Risk's per-event rule. An empty value removes the free text of a control or an outcome. `rename` changes an item's id and every reference to it, so a new id convention is one `rename` for each item. Text, tags, zones and folders are not references, so `rename` and `remove` leave them as they are. `remove` deletes an item that nothing refers to; remove a cause's or outcome's controls first, and unlink an event or outcome first. Failed commands leave the draft unchanged.

## Review and Save

```bash
node "$SKILL_DIR/scripts/lamina.mjs" check
node "$SKILL_DIR/scripts/lamina.mjs" finish "Build the flooding risk"
```

`check` validates the staged files against the schemas and checks every link. `finish` saves the changed files in one version-checked request, reads the saved model back to confirm that the server loads it, and removes the completed draft. Draft recovery, notes and offline drafting work as in the Tetra helper; see [Build models with a local agent](/docs/sightline/usage/tetra/agent-authoring).

Fields that the commands do not set, such as template definitions, objectives, Risk programs and finding links, remain available through the skill's file-level workflow.
