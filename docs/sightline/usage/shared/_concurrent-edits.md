When another program or person changes a model file while a diagram is open, Tetra and Lamina reload the model from disk. Unsaved changes in the diagram stay in place and now apply over the reloaded model.

---

## The notice

If the diagram holds unsaved changes when the reload happens, a notice appears under the toolbar:

> This model changed elsewhere. Your unsaved changes now sit over the latest version.

The notice offers two actions:

| Action | Result |
|---|---|
| **Discard changes** | Removes every unsaved change from the diagram. The model shows the saved version only. |
| **Dismiss** | Hides the notice. The unsaved changes stay. |

The notice leaves by itself when no unsaved change remains, for example after a save. It does not appear when the diagram has no unsaved changes.

In a browser page, a save made in that page does not raise the notice. Only a change made elsewhere does.

## Review before saving

The **Journal** lists every unsaved change. Read it against the reloaded model before you select **Save all**. The **Save changes** dialog that **Save all** opens does not list the changes. Where the other writer changed the same field, the value in your unsaved change is the one that saves.

## Refresh Diagram

The **Refresh Diagram** command rebuilds the diagram from disk. When the diagram holds unsaved changes, the editor asks before it discards them. The notice does not appear in that case.

---

## Per-app differences

| App | Behaviour |
|---|---|
| Tetra | The notice and both actions apply as described. |
| Lamina | The notice and both actions apply as described. |
| Metron | The Metron editors do not show the notice. |
