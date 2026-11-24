The **Models** tree in the **Sightline** view, and the sidebar of the web shell, rename, move and delete Tetra models, Lamina models, Metron packages and folders. A Metron package is a requirement package, a rule package, an attribute set package or a scheme package. A Metron assessment does not offer these entries: it sits in the folder of its package and moves and deletes with it.

A model that Sightline creates sits in a folder of its own, named after its id with the ending `.sightline`. The tree shows the model in the folder that holds that folder, and **Move…** and **Delete…** act on the model's folder as one unit. A folder name cannot end in `.sightline`, because Sightline keeps that ending for model folders.

Sightline finds models in every folder of the workspace except hidden folders, whose names start with a dot (such as `.git` or `.venv`), and `node_modules`. A model in one of those folders does not open. Move it to a visible folder.

The tree does not show the `findings` folder at the workspace root, which holds the finding registers, or any folder under it. **Move…** does not offer it as a destination. A folder named `findings` anywhere else is an ordinary folder and the tree shows it.

## Rename, Move and Delete from the Menu

1. Select **Sightline** in the activity bar.
2. Right-click a model, a Metron package or a folder in the **Models** tree.
3. Choose **Rename…**, **Move…** or **Delete…**.

| Entry | On a model | On a folder |
|---|---|---|
| **Rename…** | Changes the name the model is listed under. The model keeps its id and its files stay where they are. | Changes the folder name. Enter one folder name, not a path. Every model below the folder keeps its id. |
| **Move…** | Shows the folders of the workspace. Choose one, or **(workspace root)**. The model's directory moves into it. | Shows the folders of the workspace, without the folder itself and its subfolders. The folder moves into the one you choose. |
| **Delete…** | Asks for confirmation, then removes the model's directory. | Asks for confirmation, then removes the folder. The folder must hold no file. |

A delete cannot be undone from Sightline. Other files name a model by its id, so they keep working after a rename or a move.

## Move by Drag and Drop

Drag a model row, a package row or a folder row onto a folder row. Drop on empty space in the tree to move it to the workspace root. Select several rows first to drag them together.

A drop on a model row does nothing. A row that already sits in the target folder stays where it is. A folder cannot move into itself or into one of its subfolders.

## In the Web Shell

The sidebar opens with only **workspace** expanded, as the **Models** tree does in the **Sightline** view. Open a folder with a click or the Right arrow key, and close it with the Left arrow key. The sidebar remembers the folders you open and close in this browser, also after a refresh or a reload. A folder that is new since your last visit stays closed. A filter opens the folders of every match. When a tab becomes the active tab, the sidebar opens the folders of its model and selects the row.

The sidebar offers the same three entries.

1. Right-click a model, a Metron package or a folder in the sidebar. Alternatively, focus the row and press Shift+F10 or the Menu key.
2. Choose **Sightline: Rename…**, **Sightline: Move…** or **Sightline: Delete…**.

A dialog asks for the new name, the destination folder or the confirmation. The destination list shows the folders of the sidebar, without the folder itself and its subfolders. **workspace** is the root.

Drag a model row, a package row or a folder row onto a folder row to move it. Drop on empty space in the sidebar to move it to **workspace**. One row drags at a time.

A token with the reader role sees the three entries disabled, and no row drags. Move and Delete need the tab of the model closed, and a message names the tab to close.

## Refused Model and Folder Operations

Sightline shows one message that names what blocks the operation. Close the editor or the tab, or change the blocking file, and run the entry again.

| Message | Cause | What to do |
|---|---|---|
| Close the editor open on a model | An editor or a tab is open on the model, or under the folder, that you move or delete. | Close the editor or the tab. |
| A finding register, an assessment, a Lamina model or a requirement package still names the model | Something refers to the model's id, so a delete would leave a broken reference. A requirement package names a rule, attribute set or scheme package in its `uses` list. | Remove or change the file the message names, then delete the model. |
| The directory also holds other models | The model shares its directory with another model. | Move the other model out of the directory. |
| The model reads files outside its own directory | A move would change the path that the model uses to read them. A requirement package finds its rule, attribute set and scheme packages by kind and id, so it can move apart from them. | Keep the model's files together, or leave the model where it is. |
| The folder still holds files | A folder is deleted only when it holds no file. | Remove the files the message names. |
| The folder sits inside a model's directory, or is the findings folder | These folders keep their place, because a model's directory is one unit and the findings folder holds the registers by name. The tree does not show the findings folder, so a move into it reaches you only from a request that names it. | Leave the folder where it is. |
| A folder of that name already exists | The destination has an item of the same name. | Choose another name or destination. |

## Licence

Each operation is an edit. It needs an edit grant for the app that owns the model. A folder that holds models of both apps needs both grants. See [Licensing](/docs/sightline/usage/shared/licensing).
