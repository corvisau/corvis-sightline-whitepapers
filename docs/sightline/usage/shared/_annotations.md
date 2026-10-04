Annotation bubbles carry a `type` that controls how their `value` renders on the canvas. Tetra and Lamina share this behaviour. The set of types and the edit surfaces differ slightly. See the [Tetra](#tetra) and [Lamina](#lamina) sections.

To colour a bubble or give it an icon, see [Annotation styling](/docs/sightline/usage/shared/annotation-styling).

---

## `type: text` vs `type: markdown`

```yaml
annotations:
  - id: <string>
    type: text | markdown | statusgrid   # Tetra also accepts array and scope
    value: <string>   # for text/markdown
```

- **`type: text`**: `value` renders literally on the canvas. Newlines are preserved, and the bubble wraps them as line breaks. No Markdown syntax is applied: `**bold**` shows as literal asterisks, and `- a` shows as a literal dash.
- **`type: markdown`**: `value` renders as Markdown on the canvas. `**bold**` renders bold, and `- a` / `- b` renders as a bulleted list. A single newline in the source renders as a line break, not as a CommonMark soft-break space. A markdown value can also link to another object in the model. See [Markdown links](/docs/sightline/usage/shared/markdown-links).

The remaining types render through their own dedicated presentations. See the [Tetra](#tetra) and [Lamina](#lamina) sections.

## The props drawer Value field

The props drawer edits the Value of an existing annotation with the shared Markdown editor. The editor is the same for both `text` and `markdown` types. It previews rendered Markdown while you edit, and it matches the annotation create surface.

The Value field of a `type: text` annotation can therefore look different from the canvas bubble. The canvas keeps `text` literal, but the drawer editor always previews Markdown formatting, whatever the type. To make the canvas bubble render Markdown, set the annotation `type` to `markdown`.

## Setting a bubble's width

An annotation takes an optional `width`, in pixels. The bubble renders exactly that wide. The examples for each app are in the [Tetra](#tetra) and [Lamina](#lamina) sections.

- Without `width`, the bubble fits its content up to a maximum width. The per-link `maxWidth` sets that maximum.
- A `width` overrides every `maxWidth`. The bubble is never narrower than 72 px, so a smaller value renders at 72 px.
- `width` must be a number greater than zero. The model loader rejects zero and negative values.
- An id-only badge ignores `width`. The badge appears when the view option shows annotation ids, or when the link sets `style: id`.
- Edit `width` in the props drawer. The field is also on the create surface. Clear the field to remove `width` from the YAML.

---

## Tetra

### Types

A Tetra annotation has one of five types: `text`, `markdown`, `array`, `statusgrid` or `scope`.

### Editing a value

The bubble stays read-only on the canvas for `markdown`. Edit the value of an existing annotation in the props drawer.

### Width

```yaml
annotations:
  - id: A-0001
    type: text
    value: A long note that would otherwise stop at the default maximum width.
    width: 420
    links:
      - ref: D-00042
```

- A `type: scope` badge stays compact and ignores `width`.
- The Tabular Editor shows `width` in the shared `width` column. Channels and flows use that column for their line width.

---

## Lamina

### Types

A Lamina annotation has one of three types: `text`, `markdown` or `statusgrid`. The `statusgrid` type renders through its own chip-grid presentation.

### Editing a value

Edit the text of an annotation in the props drawer, not on the canvas bubble. The canvas right-click menu also offers **Set type** and **Set style**.

### Width

```yaml
annotations:
  - id: ann-c1
    type: text
    value: A long note that would otherwise stop at the default maximum width.
    width: 420
    links:
      - ref: C001
```
