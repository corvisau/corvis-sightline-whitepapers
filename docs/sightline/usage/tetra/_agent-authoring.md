The Sightline API skill includes an incremental Tetra helper. An agent can create one zone, group, device, network or connection at a time, review a local draft, and save the changes together. The helper handles YAML, schemas and version checks. It also includes a local image helper for diagrams larger than the agent's image input.

You need Node.js 22 or later, the complete skill folder, and an approved storage API endpoint. Your operator supplies the skill folder and endpoint. Set `SKILL_DIR` to that folder and `SIGHTLINE_API_URL` to the exact Tetra API base. If authentication is required, supply the issued token through `SIGHTLINE_API_TOKEN` using your environment's secret mechanism.

Drafts contain model data and stay on the computer running the helper. Item commands edit only that local draft. Saving sends model file contents to the configured storage API. The image helper works entirely locally and never uploads an image. Neither helper calls an LLM service.

## Create or open a model

For a new model:

```bash
node "$SKILL_DIR/scripts/tetra.mjs" new "Plant demo"
```

This creates a dedicated directory and a starter model on the server, then opens its local draft. The model id is the name converted to lowercase letters, digits and hyphens, for example `plant-demo`. Use `--parent examples` to create the directory under an existing workspace folder. For an existing model, discover its id with the skill's `models` command, then open it:

```bash
node "$SKILL_DIR/scripts/tetra.mjs" open plant-demo
node "$SKILL_DIR/scripts/tetra.mjs" list
```

The default draft is `.sightline-drafts/tetra.json`. The helper writes it; the agent does not prepare it. Supply `--draft <path>` on every command to use another location or work on several models separately. Existing drafts are protected against replacement. Use `status` to resume one. `close` removes a clean local draft and leaves the server model in place.

## Build structure, then connections

Create zones first, then their groups, devices and networks. A parent must already exist. Leaving out `--parent` places a structural item at the diagram root.

```bash
node "$SKILL_DIR/scripts/tetra.mjs" zone control "Control zone"
node "$SKILL_DIR/scripts/tetra.mjs" group rack "Controller rack" --parent control
node "$SKILL_DIR/scripts/tetra.mjs" device plc-1 "PLC 1" --parent rack
node "$SKILL_DIR/scripts/tetra.mjs" device hmi-1 "Operator panel" --parent control
node "$SKILL_DIR/scripts/tetra.mjs" network control-lan "Control LAN" --parent control
```

Each id identifies one item. Keep it stable when its label changes. Groups organise equipment; networks represent connectivity. If a group's boundary is unclear, create its devices in the known zone and move them into the group later.

Once the endpoints exist, add channels and information flows:

```bash
node "$SKILL_DIR/scripts/tetra.mjs" connect c-plc-lan plc-1 control-lan
node "$SKILL_DIR/scripts/tetra.mjs" connect c-hmi-lan hmi-1 control-lan
node "$SKILL_DIR/scripts/tetra.mjs" flow f-commands hmi-1 plc-1 "Operator commands" --path control-lan
```

The first argument to `connect` or `flow` is the connection's own id. Channels are bidirectional by default; flows go from the first endpoint to the second. Repeat `--path <id>` to describe intermediate nodes in order. Diagram coordinates are not model topology.

## Inspect and correct small pieces

```bash
node "$SKILL_DIR/scripts/tetra.mjs" find "PLC"
node "$SKILL_DIR/scripts/tetra.mjs" show plc-1
node "$SKILL_DIR/scripts/tetra.mjs" set plc-1 label "Cooling PLC"
node "$SKILL_DIR/scripts/tetra.mjs" move plc-1 control
node "$SKILL_DIR/scripts/tetra.mjs" undo
```

Use `list zone`, `list device`, or `list --parent control` for a compact inventory. `show` reads one item's properties. `set` changes a supported scalar property, `move` updates structural membership, and `endpoints` changes a channel or flow. `remove` refuses items still referenced elsewhere and containers that are not empty. Failed commands leave the draft unchanged.

The parent filter lists direct children. Inventories return up to 25 results; use `--limit` and `--offset` to read smaller pages. `find` supports the same pagination.

Use `note "Trace the line leaving tile 1"` to remember an unresolved observation. `notes` lists these observations, and `resolve-note 1` marks one resolved. Notes remain local across saves and discarded edits. They are excluded from model files. Checks and saves warn about unresolved notes, and closing a draft requires resolving them.

`undo` reverses the last staged operation. `discard` drops every local change since the last snapshot or successful save. Neither changes the server. A draft without a data slice gains an Everything slice so the saved model has a diagram view.

## Review and save

```bash
node "$SKILL_DIR/scripts/tetra.mjs" status
node "$SKILL_DIR/scripts/tetra.mjs" check
node "$SKILL_DIR/scripts/tetra.mjs" commit "Build the control network"
node "$SKILL_DIR/scripts/tetra.mjs" diagnostics
```

Resolve the problems reported by `check`. It checks schemas and known references locally; template expansion and expression membership still need server diagnostics. `commit` saves only the changed files in one version-checked API request. This is a model save, separate from a Git commit. A successful save clears the staged steps and the draft can continue with further edits.

If another editor changed the model, the save is refused and local edits remain available. Open a fresh draft under another path, inspect current items and reconcile the intended changes. After an interrupted request or uncertain server error, use `sync` before another save. It reads the server and reports whether the staged draft or the original snapshot matches. It never retries a write automatically.

Legacy models without a current diagram root need operator migration before using the incremental helper. Advanced model fields and anchored YAML edits remain available through the skill's file-level workflow.

## Read a large diagram in regions

Make a small overview to identify zones, then inspect native-resolution tiles or focused crops for labels and connections:

```bash
node "$SKILL_DIR/scripts/diagram-image.mjs" info diagram.png
node "$SKILL_DIR/scripts/diagram-image.mjs" overview diagram.png overview.png --max-size 1024
node "$SKILL_DIR/scripts/diagram-image.mjs" tiles diagram.png diagram-tiles --size 768 --overlap 128
node "$SKILL_DIR/scripts/diagram-image.mjs" crop diagram.png rack.png --x 1000 --y 600 --width 768 --height 768
```

Inputs may be PNG or JPEG. Outputs are PNG. Coordinates refer to original image pixels from the top-left corner, rather than the resized overview. Every crop must fit inside the source. Tiles overlap, include source coordinates in their filenames, and have an automatically generated bounds manifest. Files and tile directories must be new; the source is preserved. Export a PDF or SVG locally to PNG before using the helper.

Work one region at a time. Find existing items before adding equipment visible in an overlapping tile. Build local containment before connections, then add cross-region connections once both endpoints have been identified. Follow boundary-crossing lines through overlapping crops. Keep unclear observations in local notes and seek clarification instead of inventing unreadable equipment or links. Finish by resolving the notes, reviewing each zone's inventory, checking the saved model's diagnostics, and inspecting the diagram when a viewer is available.
