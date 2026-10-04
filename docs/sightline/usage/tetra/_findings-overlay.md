Tetra surfaces Metron's authored findings directly on the diagram, so an
architect can see that a node is flagged without switching apps. See
[Findings Tab](/docs/sightline/usage/metron/findings) for how findings are authored and
reviewed in Metron itself. This page covers only how they appear here.

## The badge

A small `⚠` badge appears in the corner of any Zone, Group, Device, Network,
Information, or System node that one or more Metron findings reference. It sits in the
same corner as the pruned-subtree badge and the subtree filter badge. The badge's
tooltip reads `"N finding(s)"`.

Channels and flows get the same badge on the canvas line itself. Tetra draws it
just above the midpoint of the routed path, clear of the line's label pill. It behaves like a node badge, with the same
tooltip, hover popup and three actions.

The badges need a licence that names Metron data. Without one, the diagram loads
and no badge appears. See [Licensing](/docs/sightline/usage/shared/licensing).

Findings are fetched once per model load, and refetched on **Refresh Diagram**
or an external file change. Tetra reads every `*.finding.yaml` in the workspace,
wherever the author filed it.

**Not covered: Overlay regions.** A finding can only be attached to an entity
kind that Metron can bind an assessment to. Overlays are not one of them. They
have no edit schema, never appear in the Tabular Editor, and are excluded from
the entity kinds an assessment scope can select.

## The popup

Hovering a badge opens a popup listing every finding attached to that
entity: label, a severity dot (low, medium, high or critical), and status
(open, reviewed or resolved). Three actions are available:

- **Open in Table**: switches to the Tabular Editor, filtered to exactly
  that entity (widening the row-type selection if you had narrowed it, so a
  Zone or Network shows up whichever types the table was showing).
- **Open in Drawer**: opens the entity's own props drawer with a read-only
  **Findings** section (same label/severity/status summary as the popup;
  findings are Metron-owned data and are never editable from Tetra). Every
  entity kind a finding can attach to carries that section: Device, Group,
  Zone, Network, Information, System, Channel and Flow.
- **Jump to Metron**: opens the Metron assessment on its Findings tab with
  this specific finding selected. See
  [Findings Tab](/docs/sightline/usage/metron/findings) → "Jumping in from Tetra" for what the
  tab looks like. If the assessment is already open, Metron reuses that tab
  and moves the selection to this finding. This action requires the Metron
  extension. If the extension is not installed or cannot activate, a warning
  appears instead of the jump.

## "View Findings on Diagram" command

Running **Sightline: View Findings on Diagram** (command palette, or the
toolbar button next to **Refresh Diagram**) narrows the whole canvas to just the
entities any finding references. The slice is transient and never persisted;
the **Data slice** picker shows it as `Findings (N entities)`. Picking any other
slice (or **None**) from the same picker exits back to normal browsing; the
transient entry disappears from the list. If no findings reference the
current model, an info message says so and nothing changes.

## Coverage overlay

Metron's assessment coverage is painted on the same entities through a separate,
opt-in overlay. See [Coverage overlay](/docs/sightline/usage/tetra/coverage-overlay).
