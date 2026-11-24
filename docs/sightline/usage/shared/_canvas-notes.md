A note is a post-it on the canvas of a diagram. It holds Markdown text, sits where you put it, and has the size you give it. Use a note to leave a remark on the diagram, such as a caveat on a cause or a question for the next review. Lamina and Tetra share this behaviour. A note belongs to one Risk in Lamina and to one data slice in Tetra. See the [Lamina](#lamina) and [Tetra](#tetra) sections.

A note differs from an [annotation](/docs/sightline/usage/shared/annotations). An annotation describes an object, links to it and draws a leader line to it. A note links to nothing and belongs to the canvas it sits on.

## Add a Note

1. Open the canvas that is to hold the note. In Lamina, pick a Risk in the Model Tree. In Tetra, open a data slice in the Slices view.
2. Right-click an empty part of the canvas.
3. Choose **Add note**.

The new note has its top-left corner at the pointer and measures 200 by 120 pixels. Its text box is open, so type the text straight away.

In a hosted shell, the first line of the text box already holds your name and the date and time, such as `Ada Lovelace, 2026-11-04 14:32`. The cursor sits on the line below. The time is your browser's local time. The line is ordinary text, so you can edit or delete it. A token that carries no display name shows its user id instead. A window with no signed-in user, such as the Visual Studio Code extension, starts the note empty.

## Edit the Text

Double-click a note to edit it. You can also select it and press Ctrl+E (Cmd+E on a Mac), or right-click it and choose **Edit note**.

The note turns into a text box that holds the Markdown source. Click outside the note, press Ctrl+Enter (Cmd+Enter on a Mac), or press Escape to finish. Each of these keeps what you typed.

A note renders its text in the same way as an annotation of type `markdown`. A single newline is a line break, and raw HTML does not render. Text that is longer than the note scrolls inside it.

## Move and Resize a Note

Drag a note to move it. Select a note to show its handles, and drag a handle to resize it. A note cannot shrink below 120 by 60 pixels.

## Delete a Note

Right-click the note and choose **Delete note**. You can also select the note and press Delete or Backspace.

## Change the Colour of a Note

Right-click the note, choose **Colour**, then choose one of the eight colours: Yellow, Peach, Rose, Lilac, Sky, Mint, Sage or Stone. A new note is yellow, and choosing **Yellow** returns a note to that colour.

A note keeps its colour in the light and the dark theme. The colour change is a staged change like any other, so **Save all** writes it and Undo steps it back.

## Save and Undo

A note change is a staged change like any other. **Save all** writes it to the model, **Discard all** drops it, and Undo steps it back. Undo steps one note change at a time.

## Canvas Note Position and Layout

- A note keeps its place on the canvas and does not follow an element. A layout change can therefore leave a note over an element. Move the note to clear it.
- Notes show on screen only. The PNG export, the Draw.io export, the command-line PNG tools and the reports leave them out.
- The Tabular Editor and the Model Tree do not list notes.

## Read-Only Windows

A window that cannot edit still shows the notes. **Add note**, **Edit note** and **Delete note** are unavailable there, and a selected note shows no handles. A signed-in reader in a hosted shell is the exception for Lamina: see [Notes in a hosted shell](#notes-in-a-hosted-shell).

## Notes in a Hosted Shell

In a hosted shell, each person who signs in saves new notes in a file of their own. Two people can add notes to one model at the same time, and neither save replaces the other. A save of notes never raises the notice that the model changed elsewhere.

- Everyone who opens the model sees the notes of every person.
- You change and delete only your own notes. A note that another person saved shows that person's name at its foot and has no handles. **Edit note**, **Colour** and **Delete note** show as unavailable on it.
- Notes that the model file already holds stay shared. Anyone who can edit can change or delete them, and the change saves in the model file.
- People who sign in with the same access token share one notes file.
- A reader can add, edit, move, resize and delete their own notes. Each change saves as soon as you make it, because a reader has no **Save all**. Everything else on the page stays read-only for a reader. If the server refuses a change, the note returns to what the server holds and the page shows the reason.

A window with no signed-in user, such as the Visual Studio Code extension or a local server, saves every note in the model file.

## Lamina

A note belongs to one Risk and shows only on that Risk's canvas. A Risk that holds other Risks does not show their notes.

A Risk carries notes only when the model's `risks` list defines it. **Add note** is not offered on any other Risk. **Duplicate** on a Risk copies the notes that are saved on it. In a hosted shell it copies the notes that the model file holds and leaves out the notes that people saved in their own files.

Drawing the member events of a Risk can leave a note over an element, because the note keeps its canvas position.

### Notes in a Lamina Model File

A Risk stores its notes in an optional `notes` list in the manifest.

```yaml
risks:
  - id: R001
    label: Example substation outage
    include:
      - id: E001
    notes:
      - id: note-4k2m9a
        x: -150
        y: -260
        width: 220
        height: 120
        text: "**Check** the relief valve"
```

| Field | Type | Meaning |
|---|---|---|
| `id` | Text | Names the note. Each note in a Risk has its own id. |
| `x`, `y` | Number | The top-left corner of the note, in canvas pixels from the middle of the event column. `x` grows to the right and `y` grows downwards. |
| `width`, `height` | Number above zero | The size of the note in canvas pixels. |
| `text` | Text | The Markdown. It can be empty. |
| `colour` | Text, optional | One of `yellow`, `peach`, `rose`, `lilac`, `sky`, `mint`, `sage` or `stone`. A note with no `colour` is yellow, and Sightline leaves the key out when you choose yellow. |

A note takes no other key. Lamina refuses a manifest in which a note has:

- An unknown key
- A width or height of zero or less
- An id that another note in the same Risk already uses
- A `colour` that is not one of the eight

A Sightline build from before notes arrived refuses a manifest where a Risk lists `notes`. A build from before note colours arrived refuses a note that has a `colour`. See [Risks](/docs/sightline/usage/lamina/risks) for the other fields of a Risk.

## Tetra

A note belongs to one data slice and shows only when that slice is open. Each slice lays the diagram out in its own way, so a note placed beside a zone in one slice has no meaning in another.

- **Add note** is offered while a data slice is open.
- **Duplicate** on a data slice does not copy its notes.
- Deleting a data slice deletes its notes. In a hosted shell, it deletes the notes that the model file holds and the notes you saved in your own file. A note that another person saved on that slice stops showing.
- A note keeps its canvas position when you collapse a zone or group. A collapse can therefore leave a note over an element. Move the note to clear it.

### Notes in a Tetra Model File

The manifest stores the notes of every slice in one optional `canvasNotes` list, beside `dataSlices`.

```yaml
dataSlices:
  - id: ops-view
    name: Operations
canvasNotes:
  - id: note-k3f9a1
    slice: ops-view
    x: 420
    y: -80
    width: 200
    height: 120
    text: "**Check** the firewall rules"
```

| Field | Type | Meaning |
|---|---|---|
| `id` | Text | Names the note. Each note in the model has its own id. |
| `slice` | Text | The id of the data slice that shows the note. |
| `x`, `y` | Number | The top-left corner of the note, in canvas pixels. `x` grows to the right and `y` grows downwards. |
| `width`, `height` | Number above zero | The size of the note in canvas pixels. |
| `text` | Text | The Markdown. It can be empty. |
| `colour` | Text, optional | One of `yellow`, `peach`, `rose`, `lilac`, `sky`, `mint`, `sage` or `stone`. A note with no `colour` is yellow, and Sightline leaves the key out when you choose yellow. |

A note takes no other key. Tetra refuses a save in which a note has:

- An unknown key
- A width or height of zero or less
- An id that another note already uses
- A `colour` that is not one of the eight
- A `slice` that names no data slice in the model

When it loads a model, Tetra leaves out a note whose `slice` names no data slice, and reports it in the Problems panel.

Notes need schema version 10, and a note colour needs version 11. A build from before notes arrived refuses a model that the stamp command marks at version 10. A build from before note colours arrived refuses a note that has a `colour`. See [Compatibility](/docs/sightline/upgrade/compatibility).
