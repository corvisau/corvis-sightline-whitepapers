A built-in template is a ready-made query for a `generator` rule. The rule drawer lists the built-in templates of the app in the **Templates** tab of the query editor. Tetra and Metron each ship their own list.

A template holds a program only. **Insert** writes the statement into the program and leaves the **Key parts** and **Data fields** of the rule unchanged. A saved rule holds ordinary program text and keeps no reference to the template.

## Start a rule from a built-in template

1. Open the rule drawer. In Metron, select **+ Add rule** in the rule package editor. In Tetra, open a rule in the Rules panel and set **Mode** to `generator`.
2. Select **Edit query**. The query editor opens with the **Templates** tab showing, under the heading **Built-in**.
3. Select **Insert** on a template.
4. Select **Apply**.
5. Enter the **Key parts** and **Data fields** of the rule. The statement names its key and data, and the fields must match them.

A new Metron rule starts as a `generator` on a starter query. A Tetra rule that you switch from `manual` to `generator` starts on the same starter. **Insert** replaces the starter while you have not changed the query. After a change, **Insert** adds the statement to the end of the program.

The **Built-in** list appears only while **Mode** is `generator`. A `transform` rule ends in `save(result)`, and a built-in template ends in `emitRecord`, so a `transform` rule and a `manual` rule do not show the list.

A built-in template has no **Delete** button and no rename. Metron saves the templates that you create in the manifest of the rule package, and the **Saved** list shows them under the built-in ones. The Rules panels of both apps keep no saved templates.

## Templates that ship with the apps

| Template | App | Records |
|---|---|---|
| One record for each device | Tetra and Metron | One record for each device, keyed on its id |
| One record for each zone | Tetra and Metron | One record for each zone, keyed on its id |
| Relation enters a zone | Tetra | One record for each channel or flow that crosses a zone boundary, keyed on the zone and the relation |

For another kind of entity, insert a template and change the `item.kind` test in the program.

## Differences between the apps

| Topic | Tetra | Metron |
|---|---|---|
| Where the list appears | A rule opened in the Rules panel, once its mode is `generator` | A new or opened rule in the rule package editor and in the Rules panel, once its mode is `generator` |
| Built-in templates | Three | Two |
| Saved templates | None in the Rules panel | The **Saved** list of the rule package, in the rule package editor |
