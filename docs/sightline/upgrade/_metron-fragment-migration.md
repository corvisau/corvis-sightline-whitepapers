A Metron requirement package takes its rules, attribute sets and scoring scheme only from component packages. This page applies to a package that still holds those items in its own folder.

## What Metron does with the old files

Metron does not load a rules, attribute-set or scheme file that sits in a requirement package's own folder.

Metron raises one warning for each such file. The warning names the file.

A package that selects a scheme held in its own folder does not open until the scheme moves into a scheme package. A package with no scheme has no outcomes to show.

## What the migration tool does

A migration tool moves each file into a component package. Ask Corvis for the tool that matches your release. The tool does these things:

1. It creates a rule package, an attribute-set package and a scheme package beside the requirement package.
2. It moves each file into the matching package.
3. It adds the new packages to the requirement package, so they show on its **Packages** tab.

The tool offers a dry run that shows what it would do before it changes anything.

## After the migration

1. Open the requirement package and select the **Packages** tab.
2. Check that each new package shows its rule, attribute-set or scheme entries in the **status** column.
3. Select the **Package meta** tab and check that the **Scheme** picker shows the scheme you expect.

See [Component packages](/docs/sightline/usage/metron/component-packages) for how the Packages tab resolves a package, and [Compatibility](/docs/sightline/upgrade/compatibility) for the versions that work together.
