A diagram or package editor in a browser page keeps its unsaved changes in the browser. When the page reloads or the tab closes, the next visit to the same model offers the changes back.

---

## The prompt

A **Restore unsaved changes?** dialog opens when the model loads. It states how many changes the browser kept and when it saved them.

| Action | Result |
|---|---|
| **Restore** | Stages the changes again, over the model as it is now. |
| **Discard** | Deletes the kept changes. The model shows the saved version only. |

The dialog has no close button. Choose one action to continue editing.

When the model changed after the browser saved the changes, the dialog says so. Restoring puts your changes over the latest version. Where both versions changed the same field, your value is the one that saves. Read the **Journal** before you select **Save all**. See [Changes made elsewhere](/docs/sightline/usage/shared/concurrent-edits).

## What the browser keeps

- One draft for each model, in the browser's local storage for that site
- An update to the draft after every change
- No draft once the diagram holds no unsaved changes, for example after Save all or Discard

The draft belongs to one browser. Another browser or another device does not see it. Two tabs of the same model share one draft, and the last write wins.

When the browser blocks local storage or the storage is full, the diagram works as before. Editing and Save all are unaffected, and a reload then loses the unsaved changes.

---

## Per-app differences

| App | Behaviour |
|---|---|
| Tetra | A browser page keeps a draft as described. |
| Lamina | A browser page keeps a draft as described. |
| Metron | A package page (requirement, rule, attribute-set or scheme) and an assessment page keep a draft as described. A package draft belongs to one version of the package. After **Restore**, the Evidence and Shared tabs show the restored edits, and the dashboard counts follow the saved assessment until you save. |

The Tetra, Lamina and Metron panels in VS Code keep no draft.
