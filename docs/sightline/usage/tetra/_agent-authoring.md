The Sightline API skill includes an incremental Tetra helper. An agent can create one zone, group, device, network, channel, flow or system at a time, review a local draft, and save the changes together. The helper handles YAML, schemas and version checks. It also includes a local image helper for diagrams larger than the agent's image input.

You need Node.js 22 or later, the complete skill folder, and an approved storage API endpoint. Your operator supplies the skill folder and endpoint. Set `SKILL_DIR` to that folder.

## Supply the Connection Once

Both API helpers automatically read `.sightline.env` from the working directory where the agent runs. The operator can write it when starting the server:

```dotenv
SIGHTLINE_API_URL=http://localhost:4960/api/tetra
SIGHTLINE_API_APP=tetra
SIGHTLINE_API_TOKEN=<issued-editor-token>
```

Keep the file readable only by its owner and add `.sightline.env` to the working directory's `.gitignore`. The helpers read the file directly, so the agent can run commands without copying the token into its conversation or command arguments. They parse values as data without executing shell commands or expanding variables.

Use `SIGHTLINE_ENV_FILE` to select a different file. Relative paths resolve from the working directory. The helpers check only that file; they do not search parent directories or load a generic `.env`. A missing default file is optional. A missing or unreadable explicitly selected file stops the command.

Explicit `SIGHTLINE_API_URL`, `SIGHTLINE_API_APP` and `SIGHTLINE_API_TOKEN` environment values override their file values. An explicitly empty token disables the file token. Without a configured URL, the fallback is `http://localhost:4960/api/tetra`. A token in the file must be accompanied by its URL. Changing the environment URL to another server requires an explicit token for that server or a different connection file. A different app path on the same server can reuse the file token.

The file is reread on each command. After the operator updates it, the next command uses the new connection. The server still checks the token's permissions and its licence before allowing saves.

Drafts contain model data and stay on the computer running the helper. Item commands read the server version before editing the local draft. Saving sends model file contents to the configured storage API. The image helper works entirely locally and never uploads an image. Neither helper calls an LLM service.

## Give the Agent Modelling Conventions

Write your organisation's modelling conventions at the root of the working directory: the instructions for all models in `SIGHTLINE.md`, and the Tetra conventions in `SIGHTLINE_TETRA.md`, such as zone types, tags, label formats, and what to model and what to leave out. The agent reads these files before each task on a Tetra model and follows them. Without the files, the agent follows only the method. When `SIGHTLINE.md` also names a security standard, such as `SIGHTLINE_SECURITY.md`, the agent still records what the source shows, even where the system breaks the standard, and reports each breach it sees with the requirement it breaks.

## Create or Open a Model

For a new model:

```bash
node "$SKILL_DIR/scripts/tetra.mjs" new "Plant demo"
```

This creates a dedicated directory and a starter model on the server, then opens its local draft. The model id is the name converted to lowercase letters, digits and hyphens, for example `plant-demo`. Use `--parent examples` to create the directory under an existing workspace folder. For an existing model, discover its id with the skill's `models` command, then open it:

```bash
node "$SKILL_DIR/scripts/tetra.mjs" open plant-demo
node "$SKILL_DIR/scripts/tetra.mjs" list
```

The default draft is `.sightline-drafts/tetra.json`. The helper writes it; the agent does not prepare it. Supply `--draft <path>` on every command to use another location or work on several models separately. `open` can refresh a clean draft for the same model and API. Drafts with unsaved edits, unresolved notes or an uncertain save are protected against replacement. `close` removes a clean local draft and leaves the server model in place.

### Resume in a New Session

Old draft files can be saved snapshots or unfinished work. Start by identifying them:

```bash
node "$SKILL_DIR/scripts/tetra.mjs" drafts
node "$SKILL_DIR/scripts/tetra.mjs" status --draft .sightline-drafts/tetra.json
```

`drafts` lists local summaries without contacting a server or printing model file contents. An optional directory argument inventories another location. Unsupported files are reported and left alone. `status` then checks the relevant draft against its server.

The output names the model, API, draft path and baseline version. Its `lifecycle` reports whether local files are `clean`, `modified` or `uncertain`, with changed files and unresolved notes. Confirmed saves also record their message, version and local confirmation time in `lastSave`. Older drafts may lack this receipt; the helper still compares their files with the baseline to identify unsaved changes.

Follow `lifecycle.nextAction`, which supplies a command, argument array and explanation. A stale clean snapshot can be refreshed by opening the same model at the same path. A stale draft containing edits needs preservation and reconciliation in a separate draft. An uncertain save requires `sync` first. Review inherited edits and notes against the current task before saving or discarding them.

## Build Structure, Then Connections

Create zones first, then their groups, devices and networks. A parent must already exist. Leaving out `--parent` places a structural item at the diagram root.

```bash
node "$SKILL_DIR/scripts/tetra.mjs" zone control "Control zone" --type OT-L1 --slt 2
node "$SKILL_DIR/scripts/tetra.mjs" group rack "Controller rack" --parent control
node "$SKILL_DIR/scripts/tetra.mjs" device plc-1 "PLC 1" --parent rack
node "$SKILL_DIR/scripts/tetra.mjs" device hmi-1 "Operator panel" --parent control
node "$SKILL_DIR/scripts/tetra.mjs" network control-lan "Control LAN" --parent control --protocol ethernet,ip
```

`zone` takes the zone type with `--type`, the SL-T with `--slt` and the Threat Potential with `--tp`. `network` takes its address ranges with `--address` and its default protocol stack with `--protocol`. Every command that creates an item takes tags with `--tag`, given once per tag or as one comma-separated value, such as `--tag scada,critical`. Each id identifies one item. Keep it stable when its label changes. Groups organise equipment; networks represent connectivity. If a group's boundary is unclear, create its devices in the known zone and move them into the group later.

The UI's "add network to device" adds a channel from the device to the network. The device and network keep their structural parents. Use `attach` for this ordinary network membership:

```bash
node "$SKILL_DIR/scripts/tetra.mjs" attach plc-1 control-lan
node "$SKILL_DIR/scripts/tetra.mjs" attach hmi-1 control-lan
node "$SKILL_DIR/scripts/tetra.mjs" flow f-commands hmi-1 plc-1 "Operator commands" --path control-lan
```

Each attachment is a channel with an id like `C-plc-1-control-lan` and `routing: direct`, which carries the network's default protocol stack, or `ip` when the network has none. `show` lists a device's `networks` and a network's `devices`, names the channels in `attachmentChannels`, and marks such a channel `attachment: true`. Repeating `attach` leaves an existing attachment alone, and `connect` refuses a second network channel between a device and a network it is already on. `detach` removes the attachment and its channel. A virtual or hosting channel between a device and a network is not an attachment, and `detach` leaves it alone. A device that still lists the retired `connections` makes `attach` and `detach` stop; run the synthetic-channels migration on the model first.

To give an attachment its own id, protocol or direction from the start, create it with `connect` in place of `attach`, for example `connect c-plc-lan plc-1 control-lan --protocol ethernet,ip`.

The first argument to `connect` or `flow` is the relationship's own id. Channels are bidirectional by default; flows go from the first endpoint to the second. `--path` lists the intermediate nodes in order. A path holds networks, devices, zones or groups, never a channel. Diagram coordinates are not model topology.

## Describe Channels, Flows and Systems

The options below set the values that [Tetra entities](/docs/sightline/methodology/tetra-entities) and [Tetra channels and flows](/docs/sightline/methodology/tetra-channels-and-flows) define.

```bash
node "$SKILL_DIR/scripts/tetra.mjs" device host-1 "Server host" --parent control
node "$SKILL_DIR/scripts/tetra.mjs" device vm-1 "Control VM" --parent control
node "$SKILL_DIR/scripts/tetra.mjs" connect c-host host-1 vm-1 --type hosting
node "$SKILL_DIR/scripts/tetra.mjs" information level "Tank level" --owner plc-1
node "$SKILL_DIR/scripts/tetra.mjs" flow f-level level vm-1 "Level polling" --type function --initiation programmatic --protocol modbus --path control-lan
node "$SKILL_DIR/scripts/tetra.mjs" system s-tank "Tank control" --member plc-1,vm-1
node "$SKILL_DIR/scripts/tetra.mjs" annotate plc-1 "Assumed: the firmware is current."
```

- `connect --type` sets the channel type: `network`, which is the default, `virtual` or `hosting`. A hosting channel takes no protocol. `connect` takes several end nodes, comma-separated, as `flow` does, for example `connect c-host host-1 vm-1,vm-2 --type hosting` for one host and its virtual machines.
- `--protocol` takes the stack from the lowest layer up, as one comma-separated value, such as `ethernet,ip,tcp`. The helper writes each entry in lowercase and joins words with a hyphen, so `OPC UA` becomes `opc-ua`.
- `flow --type` sets the flow type: `function`, `system` or `security`. `--initiation` sets who starts the exchange: `interactive` or `programmatic`. `--direction` sets the flow's `infoDirection` and takes `a-to-b`, `b-to-a` or `bidirectional`. `--initiator` takes `a`, `b` or `both` and names the end that opens the exchange. Leave it out when the source does not say. `--criticality` takes `critical`, `important` or `normal`.
- `flow` takes several end nodes as one comma-separated value, as in `flow f-logs plc-1 historian,scada "Logs"`. It stays one flow, and the canvas draws the shared path once with a short line to each end. Write one flow for each route when the ends need different routes.
- `system` creates a system. `--member` lists its zones, groups, devices and networks.
- `annotate` attaches a note to an item. Name several items, separated by commas, for one note on all of them. The note is a `text` annotation unless `--type markdown` is given. Its id is `note-1`, `note-2` and so on, unless `--id` gives one. `set <note> value <text>` changes the text, and `remove <note>` deletes the note. An item that has a note cannot be removed until the note is removed. Other annotation types hold structured values, which the agent writes through the model files.
- A list option takes one comma-separated value, as in `--path control-lan,fw-1` or `--member plc-1,vm-1`. Giving the option once for each value also works.

The helper refuses a value outside these lists before it stages the edit, and names the values it accepts. Each staged edit prints the fields it set, so the agent can compare them with its source.

## Set Default or View-Specific Order

Use `order <parent>` to inspect a zone or group's default membership order. Use `root` for the hidden diagram root. Supply every authored child id once to change that order directly, while keeping each child in its parent:

```bash
node "$SKILL_DIR/scripts/tetra.mjs" order control
node "$SKILL_DIR/scripts/tetra.mjs" order control control-lan rack hmi-1
node "$SKILL_DIR/scripts/tetra.mjs" order control --slice all
```

This edits the same default as **Advanced → File order**. Definitions may be organised by type in YAML; the parent's mixed `members` sequence sets its default child order. `undo` reverses the reorder in one step.

View ordering takes precedence over this default. Inspection lists `fileOrder`, explicit slice overrides and warnings. With `--slice`, it also reports `effectiveOrder` and the `orderSource`, including the base slice supplying an inherited order. The effective list covers authored children before view filtering; expression-generated membership and the final canvas still need viewer verification. Information badges retain their own slots in a view order.

Use `list slice` to discover view ids, names and their base slices. It accepts the same pagination options as other inventories.

- Add `--slice <id>` to a reorder to change only that view.
- Use `order <parent> --slice <id> --reset` to clear its local order. It can then inherit an order from its base slice. Inspection names that source.
- Add `--clear-view-orders` to a default reorder when every view should follow the file default. This clears only that container's ordering overrides across all slices, keeping their other layout fields.

After saving, open a fresh verification draft under another path and inspect the target slice's order. Check the viewer for appearance. Expression-generated children cannot be individually arranged through the authored order; their expression entries follow the listed members.

## Inspect and Correct Small Pieces

```bash
node "$SKILL_DIR/scripts/tetra.mjs" find "PLC"
node "$SKILL_DIR/scripts/tetra.mjs" show plc-1
node "$SKILL_DIR/scripts/tetra.mjs" set plc-1 label "Cooling PLC"
node "$SKILL_DIR/scripts/tetra.mjs" tag plc-1 safety
node "$SKILL_DIR/scripts/tetra.mjs" move plc-1 control
node "$SKILL_DIR/scripts/tetra.mjs" undo
```

Use `list zone`, `list device`, or `list --parent control` for a compact inventory. `show` reads one item's properties. `set` changes a supported property, such as `set control type OT-L2` or `set c-host type virtual`. On a zone, `type` is the zone type; on a channel, it is the channel type. `set <system> members a,b` replaces a system's members, and `set` also changes a protocol stack. `set <flow> path a,b` replaces the intermediate nodes of a channel or flow, and `set <flow> path none` removes them. `set <network> addressing a,b` replaces a network's address ranges. `move` updates structural membership, and `endpoints` changes the ends of a channel or flow. It takes several end nodes as one comma-separated value, and that list replaces the ends. On an item with several ends, a single end is refused so that no end is lost by accident: list every end to keep, or add `--single` to leave only that one. `remove` refuses items still referenced elsewhere and containers that are not empty. Failed commands leave the draft unchanged.

Every item can carry tags. `set <id> tags a,b` replaces an item's tags, and `set <id> tags none` removes them. `tag <id> a,b` adds tags and `untag <id> a,b` removes them; the item's other tags stay. The helper trims each tag and drops repeats. It keeps the case, because a tag filter matches case exactly, so `SCADA` and `scada` are different tags. `show` and each staged edit list the item's tags. A tag is never treated as an id, so `rename` and `remove` leave it alone even when it matches one.

Items can carry your own named properties. `property <id> <name> <value>` sets one and leaves the others, and `none` removes it. A whole number and `true` or `false` keep that type. `--list` takes a comma-separated list, and `--text` keeps a value such as `3` as text. The commands that create a zone, group, device, network, channel, flow or system take `--property name=value`, as in `device plc-1 "PLC 1" --property owner=ops`. A property is never treated as an id, so `rename` and `remove` leave it alone.

`rename <id> <new-id>` changes an item's id and every reference to it: its parent's membership, attachments, endpoints, paths, system members and view settings. Labels and descriptions stay as they are. The command refuses an id that is already in use. When an expression in a view mentions the old id, the result names it so that you can change it by hand. A new id convention for a whole model is one `rename` for each item, saved together.

`query` searches on any field and chooses what each result shows. `--missing <field>` keeps items without a field, and `--has <field>` keeps those with it. `--where` takes `field=value`, `field!=value`, `field~text`, or `>`, `<`, `>=` and `<=` with a number. `--show` picks the fields of each result, and `--show all` gives every field. For example, `query device --missing image --show label,description` lists the devices with no image. A field can be a path, such as `properties.owner`. `fields` lists the fields that items hold, with a count and an example of each. A field that no item holds gives a warning, so a misspelt name does not look like an answer.

The parent filter lists direct children. `--tag scada` keeps only the items that carry that tag; give several tags to keep the items that carry all of them. Inventories return up to 25 results; use `--limit` and `--offset` to read smaller pages. `find` supports the same tag filter and pagination.

Use `note "Trace the line leaving tile 1"` to remember an unresolved observation. `notes` lists these observations, and `resolve-note 1` marks one resolved. Notes remain local across saves and discarded edits. They are excluded from model files. Checks and saves warn about unresolved notes, and closing a draft requires resolving them.

`undo` reverses the last staged operation. `discard` drops every local change since the last snapshot or successful save. Neither changes the server. A draft without a data slice gains an Everything slice so the saved model has a diagram view.

## Review and Save

```bash
node "$SKILL_DIR/scripts/tetra.mjs" status
node "$SKILL_DIR/scripts/tetra.mjs" check
node "$SKILL_DIR/scripts/tetra.mjs" finish "Build the control network"
```

Resolve the problems reported by `check`. It checks schemas and known references locally; template expansion and expression membership still need server diagnostics. `finish <message>` saves changed files in one version-checked API request, reads saved diagnostics and removes the completed draft when both `diagnostics` and `problems` are empty. A save message is required when edits remain unsaved. This is a model save, separate from a Git commit.

For an intermediate checkpoint, use `commit <message>`. It records the confirmed save, clears staged steps and leaves the draft open for further edits. `finish` without a message can close an already saved draft after checking diagnostics, without another write.

Unresolved notes and uncertain saves block finishing. When diagnostics are unavailable or need review, the draft stays in place even if the model was saved. Correct the problems and finish again, or explicitly `close` a clean draft after reviewing intentional diagnostics. A retry after a confirmed save performs reads unless new edits have been staged. The helper never deletes a directory of drafts or removes files because of their age.

Each item edit checks the live model version before staging. Inspections and `status`/`check` return `freshness`, including the server version and changed files when the draft is stale. A UI save of a view or data slice changes this same version. The next command detects it; the helper does not monitor in the background.

A stale draft or an unavailable version check stops further edits. Existing work stays intact and can still be inspected, undone or discarded locally. Follow the returned action: refresh a clean snapshot in place, or preserve unsaved work and reconcile it with a separate current draft. The final save retains its version guard for changes made after the preliminary check.

For deliberately disconnected drafting, `--offline` skips pre-edit and inspection reads and returns an `unchecked` warning. A commit always checks the server, including with this option. It keeps the draft's original base version.

After an interrupted request or uncertain server error, use `sync` before another save. It reads the server and reports whether the staged draft or the original snapshot matches. It never retries a write automatically.

Legacy models without a current diagram root need operator migration before using the incremental helper. Advanced model fields and anchored YAML edits remain available through the skill's file-level workflow.

## Read a Large Diagram in Regions

Make a small overview to identify zones, then inspect native-resolution tiles or focused crops for labels and connections:

```bash
node "$SKILL_DIR/scripts/diagram-image.mjs" info diagram.png
node "$SKILL_DIR/scripts/diagram-image.mjs" overview diagram.png overview.png --max-size 1024
node "$SKILL_DIR/scripts/diagram-image.mjs" tiles diagram.png diagram-tiles --size 768 --overlap 128
node "$SKILL_DIR/scripts/diagram-image.mjs" crop diagram.png rack.png --x 1000 --y 600 --width 768 --height 768
```

Inputs may be PNG or JPEG. Outputs are PNG. Coordinates refer to original image pixels from the top-left corner, rather than the resized overview. Each output has a ruler on its top and left edges, labelled in original image pixels, so a position read from an overview, tile or crop can be used directly as a crop coordinate. Add `--no-ruler` to leave it off. A long, thin crop with `--max-size` shows a long line from end to end. Every crop must fit inside the source. Tiles overlap, include source coordinates in their filenames, and have an automatically generated bounds manifest. Files and tile directories must be new; the source is preserved. Export a PDF or SVG locally to PNG before using the helper.

Work one region at a time. Find existing items before adding equipment visible in an overlapping tile. Build local containment before connections, then add cross-region connections once both endpoints have been identified. Follow boundary-crossing lines through overlapping crops. Keep unclear observations in local notes and seek clarification instead of inventing unreadable equipment or links. Finish by resolving the notes, reviewing each zone's inventory, checking the saved model's diagnostics, and inspecting the diagram when a viewer is available.

## Write and Run Rules

The agent writes the rules a model runs with the rules helper. It creates a rule package, adds it to the model's `uses:` list, and writes each rule as a small YAML file with its program as a block:

```bash
node "$SKILL_DIR/scripts/rules.mjs" create "Plant checks"
node "$SKILL_DIR/scripts/rules.mjs" use plant-a plant-checks
node "$SKILL_DIR/scripts/rules.mjs" test plant-a plant-checks cross-zone-flow.yaml
node "$SKILL_DIR/scripts/rules.mjs" save plant-a plant-checks cross-zone-flow.yaml
node "$SKILL_DIR/scripts/rules.mjs" run plant-a
```

`fields` lists every row of the model with the fields a rule reads. `lint` checks a rule file for the expression forms that do not work as they read, without a server; `test` and `save` run the same check and list its warnings. `test` runs one rule without writing and lists the records it would create, change or close, with any problem in the program. `save` tests the rule again and refuses it while the test reports an error, then adds the rule to the package or replaces the rule with the same id. `run` writes the records; a second run of an unchanged model changes nothing. `records` lists what each rule wrote, and `show` prints a saved rule for editing.

A rule with `hide: true` writes diagnostics, which the Problems view shows. A rule with `input: records` rolls up the records of the rules in its `dependsOn`. The helper takes a rule's key parts and data fields from its `emitRecord` statement.

The package is created and edited through the Metron storage server on the same address, and the rules run on the Tetra one. Behind the web shell both are found from the one connection file. With separate servers, the connection file also names the Metron address as `SIGHTLINE_RULES_API_URL`. When the connection uses a token, that address must be on the same origin as the Tetra one, so the token goes to no other server.
