A Metron rule can create records from the assessed pairs and attributes of one Tetra model. The Records panel lists the rules, tests an edit and runs them. The panel also lists each finding that a person creates on the [Findings tab](/docs/sightline/usage/metron/findings), a record of the manual rule `finding`.

## Which rules run

A run covers one Tetra model. It runs the rules of the rule packages that the requirement packages of that model name. A requirement package counts when an assessment binds it to the model.

To make the rules of a rule package run for a model, follow these steps:

1. Open the requirement package. Select the **Packages** tab.
2. Select **Add package**. Choose the rule package in the dialog.
3. To check the domain, open the rule package. Select its **Package meta** tab. The **Domain** field must read `metron`.

The command **Metron: New Rule Package** creates a package that runs in Metron unless you choose another domain in its form.

A rule runs only when its rule package has the domain `metron` and the rule has a program. Every rule in the package runs in that domain. A scope rule or an assessment rule creates no record, so a run leaves it alone.

A model with no assessment runs no rule. An assessment that cannot load adds a warning to the run. When no assessment of the model loads, the run skips each package with a rule that reads pairs or attributes, and reports the reason. The existing records stay as they are.

## Write a rule

Create and edit rules in the rule package editor. The **Rules** tab is a grid, and the rule drawer holds every create and edit.

1. Open the rule package. Select the **Rules** tab.
2. Select **+ Add rule**. The drawer opens as **New rule**.
3. Enter a **Rule id** and a **Title**.
4. Choose a **Mode**. A rule that creates records uses `generator`.
5. Choose an **Input**. With `model`, the rule reads one row for each entity of the Tetra model. With `records`, the rule reads one row for each record of the rules in **Depends on**.
6. Select **Edit query**. The builder opens on a starter statement for the mode. Select **Apply** to write it as the program, and change it as needed.
7. Select **Save**, then **Save all**.

The drawer starts a new rule as a `generator` on a starter query. **Edit query** opens on the built-in templates, and **Insert** replaces the starter with the template you choose. See [Rule templates](/docs/sightline/usage/shared/rule-templates).

A rule that reads the assessed pairs of a model, or the assessed attributes of a pair, keeps its **Input** when you edit it. A record about one outcome comes from the first kind of rule. A record about one attribute comes from the second kind. An attribute row exists only for an attribute that the assessor has assessed.

### Links

A record stores what it depends on as links. A link names an entity of the Tetra model, a requirement, an assessed pair or an assessed attribute. The record stores the version of the entity or the requirement. For a pair or an attribute, the record stores a hash of the assessed state.

A change to the notes or the evidence of a pair does not move a pair link or an attribute link.

## Open the panel

Right-click an assessment in the Models tree and select **Metron: Open Records**. The panel opens for the Tetra model of the assessment. The command **Metron: Open Records** in the palette asks which assessment.

Each model has one panel, and a second request reveals the open one. Two assessments of the same model open the same panel. The browser version of Metron does not show this panel.

## The Rules tab and the Records tab

The panel has the two tabs and the controls of the Tetra Rules panel, including **Columns**, **Filter** and **Group** on both tabs. See [Rules and records](/docs/sightline/usage/tetra/rules-and-findings) for the drawer, **Test rule**, **Save** and **Run rules**. The records list is read-only; accept a finding in the assessment's **Findings** tab. Two differences apply in Metron:

- The tab lists the rules of the rule packages that the requirement packages name. The empty message says that no rule package is named.
- The query builder offers the standard functions. A generator rule creates records with `emitRecord`.

A rule save changes only that rule in its package. Other models and requirement packages that use the package see the change.

## Run rules

**Run rules** shows how many records a run creates and how many it closes. **Confirm** writes them. A run never deletes a record and never changes the status, severity or description that a person set.

A run covers each rule package once for the model. A package with a program error is skipped, and its records stay unchanged.

### Manual records

A run also refreshes the links of manual records. It resolves each link again. When a version or a hash moved, the run records the new value and the record shows **Source changed**. The data and the set of links stay as the person set them.

A link under a scope that the run does not hold stays as stored, and the run reports a warning. A link to an id that no longer exists shows as missing, which counts as a change.

## Licence

A run and a rule save need an edit licence for Metron. A refused action shows its reason in the panel. See [Licensing](/docs/sightline/usage/shared/licensing).
