Annotation bubbles carry a **Type** that controls how their text renders on the canvas. Tetra and Lamina share this behaviour. The set of types and the edit surfaces differ slightly. See the [Tetra](#tetra) and [Lamina](#lamina) sections.

To colour a bubble or give it an icon, see [Annotation styling](/docs/sightline/usage/shared/annotation-styling).

---

## Text and Markdown types

- `text`: the value renders literally on the canvas. Newlines are preserved, and the bubble wraps them as line breaks. No Markdown syntax is applied: `**bold**` shows as literal asterisks, and `- a` shows as a literal dash.
- `markdown`: the value renders as Markdown on the canvas. `**bold**` renders bold, and `- a` / `- b` renders as a bulleted list. A single newline renders as a line break, not as a CommonMark soft-break space. A Markdown value can also link to another object in the model. See [Markdown links](/docs/sightline/usage/shared/markdown-links).

The remaining types render through their own dedicated presentations. See the [Tetra](#tetra) and [Lamina](#lamina) sections.

## The props drawer Value field

The props drawer edits the **Value** of an existing annotation with the shared Markdown editor. The editor is the same for both `text` and `markdown` types. It previews rendered Markdown while you edit, and it matches the annotation create surface.

The **Value** field of a `text` annotation can therefore look different from the canvas bubble. The canvas keeps `text` literal, but the drawer editor always previews Markdown formatting, whatever the type. To make the canvas bubble render Markdown, set the annotation **Type** to `markdown`.

## Setting a bubble's width

The **Width** field in the props drawer sets the bubble width in pixels. The bubble renders exactly that wide. The field is also on the create surface.

- With **Width** empty, the bubble fits its content up to a maximum width. The **Max width** setting on each link sets that maximum.
- A **Width** overrides every **Max width**. The bubble is never narrower than 72 px, so a smaller value renders at 72 px.
- **Width** must be a number greater than zero. The model loader rejects zero and negative values.
- An id-only badge ignores **Width**. The badge appears when the view option shows annotation ids, or when the link **Style** is `id`.
- Clear the field to return the bubble to its content size.

## One bubble for several targets

An annotation linked to several owners shows one bubble per owner by default. To draw a single bubble instead, set **Bubble** to `shared` in the props drawer. The setting is also on the create surface, in Tetra and in Lamina.

- The shared bubble has a leader line to each target. It sits beside the first target that is visible, and it takes the placement of that link: **Side**, offsets and **Distance**.
- Alt+right-click on the shared bubble opens the nudge panel for that first visible link. A change there moves the whole bubble.
- A target that the active data slice hides loses its leader line. The bubble stays while any target is visible, and it disappears when every target is hidden.
- When the first target is hidden, the bubble moves to the next visible link and takes that link's placement.
- An annotation with one link looks the same in both modes. A `scope` annotation ignores **Bubble** and keeps one badge per zone.
- Set **Bubble** back to `perLink`, or clear it, to return to one bubble per owner.

## Moving a nudge panel

The Alt+right-click panels have a title bar that names the panel. Drag the title bar to move the panel off the part of the diagram you are adjusting.

- The panel opens where you last left it. One position serves every nudge panel in both apps.
- Double-click the title bar to return the panel to its default place. That place is below and right of the click.
- Focus the move button in the title bar. An arrow key moves the panel 10 px, and Shift with an arrow key moves it 50 px.
- Press Home on the move button to return the panel to its default place.
- The close button in the title bar closes the panel, as Escape does.

The application stores the position on this computer, not in the model. A saved position that no longer fits the window is pulled back inside it. Lamina has the annotation placement panel only. Tetra also has the segment nudge and Container Properties panels.

## Where a note goes when you set no placement

A note that has no placement set takes the first free slot around its owner. The first note sits above the owner. A later note on the same owner moves to the next slot that does not overlap a note already placed.

- The slots are tried in this order: top middle, bottom middle, right middle, left middle, top right, top left, bottom right, bottom left, right top, right bottom, left top, left bottom.
- Notes are checked in the order they were added to the model.
- A note that you have placed stays where you put it; other notes do not move it.
- When every slot overlaps a note, the note keeps the top middle slot.
- The check covers notes only. A note can still sit over a node, a zone or a flow.
- The model does not store the slot. The slot is worked out each time the diagram opens.
- The check uses the size the bubble is estimated to have before it renders. A bubble much larger than its estimate can still overlap a neighbour, so place one of the two notes with its placement panel.

---

## Tetra

### Types

A Tetra annotation has one of five types: `text`, `markdown`, `array`, `statusgrid` or `scope`.

### Editing a value

The bubble stays read-only on the canvas for `markdown`. Edit the value of an existing annotation in the props drawer.

### Deleting an annotation

1. Right-click the annotation bubble and choose **Delete…**.
2. Read the confirmation. It states how many owners the annotation is linked to.
3. Choose **Delete**.

The delete removes the annotation and all of its links, whichever bubble you right-click. An annotation linked to several owners shows one bubble per owner, or one shared bubble when **Bubble** is `shared`. The delete removes the whole annotation either way. The delete stays pending like any other until you save, and undo or **Discard** reverses it. The menu entry is disabled when the model is read-only.

### Width

- A `scope` badge stays compact and ignores **Width**.
- The Tabular Editor shows the width in the shared **Width** column. Channels and flows use that column for their line width.

---

## Lamina

### Types

A Lamina annotation has one of three types: `text`, `markdown` or `statusgrid`. The `statusgrid` type renders through its own chip-grid presentation.

### Editing a value

Edit the text of an annotation in the props drawer, not on the canvas bubble. The canvas right-click menu also offers **Set type** and **Set style**.

### Shared bubble

A data slice in Lamina changes which targets appear, but it does not move annotations. A shared bubble stays beside the first visible target, and it moves to the next visible target when a slice hides the first.
