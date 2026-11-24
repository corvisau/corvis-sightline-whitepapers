A sync rule merges records from a connected dataset into a consolidated dataset. Running one is a single command. Understanding what it produced is the part worth reading.

**Start in your [runtime instance](/docs/prism/install/runtime-instance).** The sync command resolves its rule against the current directory.

```bash
cd acme-central
```

## Run the Rule

Name the rule by its id.

```bash
npx prism-sync aws-mock-to-consolidated
```

**Expected output.** A start line naming the rule, then a line naming where the rule was loaded from. Per-change lines follow, each showing the mode, the priority and the status. A summary of what was applied closes the run.

### Rule Lookup Order

The command looks in two places, in order:

1. `rules/<rule-id>.json`, relative to your current directory.
2. The rule storage collection, if no such file exists.

It prints which one it used. Keep your rule files. The file path is the one that works reliably: see [If the rule is not found](#if-the-rule-is-not-found).

A rule that exists but is disabled stops the run with a message saying so.

## Auto-Apply Against Pending Review

The mode on each field rule decides whether you have more work to do.

Every field rule carries a mode:

| Mode | What happens when the sync runs |
|---|---|
| `auto` | The change is applied immediately, where priority permits. |
| `review` | The change is recorded and left **pending**. Nothing is written yet. |

**A sync that reports pending changes has not failed.** It has queued them.

Priority still governs an `auto` change. A lower-priority source does not overwrite a higher-priority value simply because its rule says `auto`.

## Direction Follows the Target, Not the Rule

There is one rule shape for both directions. A reverse rule is run by the same command as a sync rule.

**The direction comes from where the target table lives.** A local target merges into a consolidated dataset. A remote target enqueues writes to that target's outbox instead, for an edge to drain later.

So you do not choose a direction when you run a rule, and there is no separate reverse-sync command. You choose a target when you write the rule. [Run a write-back](/docs/prism/usage/write-back) covers what happens after a remote target is enqueued.

## If the Rule Is Not Found

The command reports where it looked. Two causes are common, and the second one is not obvious.

**The file is not there.** You are in the wrong directory, or the rule id does not match the filename. The command prints the exact path it tried.

⚠️ **The rule was migrated into storage.** `prism-migrate-rules` writes rules into a `rules` collection, but this command's storage fallback reads a `sync_rules` collection. A rule that exists only in storage after a migration is therefore not found here.

**Keep the `rules/*.json` files in your instance.** They are the path this command resolves first, and the one that does not depend on that mismatch.

## See What the Sync Produced

The run summary tells you the counts. It does not tell you whether the pending changes are right.

Every change left pending is waiting in a changeset. Review it, decide each change, and apply what you accept: [Review and apply changesets](/docs/prism/usage/review-changesets).
