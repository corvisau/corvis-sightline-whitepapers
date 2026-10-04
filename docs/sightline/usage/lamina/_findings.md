Lamina reads the findings that Metron authors and never writes a finding's own fields. It writes only a promotion pointer (see the Promoting a finding into a cause section). This lets a user turn an architectural condition that Metron flagged into a risk-register cause, and see which findings a cause was promoted from.

## The Findings inbox

Click **Open Findings Inbox** in the toolbar. The inbox lists every finding in the workspace that has not yet been promoted into a cause. It glob-scans every `*.finding.yaml` register, as the Metron Findings tab does, and skips any register inside a dependency, build-output or version-control folder. Each row shows the label and description of one un-promoted finding.

## Promoting a finding into a cause

Click **Create cause from finding** on an inbox row. The inbox closes and the
props drawer opens on its new-cause surface, prefilled with the finding's
label and description. Adjust any field. Pick the destination file. Press **Create**. Before that, nothing is staged or written. **Create** then does two things:

1. Create stages the new cause into the diagram's pool of unsaved changes, so it commits on
   the next **Save all**, like any other new entity.
2. Create also writes `causeRef` onto the finding's own source file right away,
   marking it promoted. Reopening the inbox no longer lists the finding.

**Cancel** closes the drawer and leaves the finding un-promoted in the inbox. If
the `causeRef` write fails, the cause stays staged and a message names the
finding that was not linked.

## The Findings section on a promoted cause

Opening the props drawer on a cause shows a **Findings** section that lists every finding promoted into it. Lamina resolves the list at read time by scanning every finding register for a `causeRef` that matches the cause's id. The cause itself does not store the list. The section is read-only: it shows the linked finding's id and label, with no checkbox and no unlink control. Only promotion (see the Promoting a finding into a cause section) creates a link.

A cause with no linked findings shows the section's empty state.

## Where the link is stored

`causeRef` on the finding is the only persisted pointer. Lamina computes the
cause's Findings section from it on every read. For how the apps share findings,
see the [Cross-app workflows](/docs/sightline/architecture/cross-app-workflows) page.
