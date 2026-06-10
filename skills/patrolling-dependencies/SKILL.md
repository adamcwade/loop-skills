---
name: patrolling-dependencies
description: Use when project dependencies drift out of date, when security advisories and CVE audits should be checked on a schedule, or when routine version bumps should arrive as small tested pull requests instead of a quarterly big-bang upgrade.
---

# Patrolling Dependencies

## Overview

Check dependencies on a schedule and turn needed updates into small, tested, reviewable pull requests. The loop makes upgrades boring: one package at a time, tests prove each bump, and a human merges.

Core principle: an upgrade PR the loop cannot test is a liability, not a contribution. No green tests, no PR.

## When to Use

- Dependencies routinely fall behind until an upgrade becomes a project
- Security audits (`npm audit`, `pip-audit`) should run on a schedule, not when someone remembers
- You want patch and minor bumps flowing as steady small PRs

When NOT to use: repos without a meaningful test suite (the loop cannot verify safety; add tests first), or where Dependabot or Renovate already runs (patrol their PRs instead: rebase, read changelogs, summarize risk).

## One Iteration

1. Read the ledger, `.loop/dep-patrol.md`. If it does not exist, create `.loop/` and the file with a baseline entry: open patrol PRs found via `gh pr list --label dependencies`, an empty deferred list, and no last audit time.
2. Check the stop conditions table below against the ledger. Any row matches: do its action.
3. Audit first: `npm audit --json` (or `pip-audit`). Any new advisory beats version drift in priority.
4. Pick ONE update, by priority: security fix, then patch, then minor. Majors are flagged in the report with a changelog summary, never auto-PRed.
5. Skip it if it is already deferred, already has an open patrol PR, or pins a known incompatibility (record the reason).
6. On a fresh branch: bump that one package, install, run the FULL test suite and build.
   - Green: open a PR titled `chore(deps): bump <pkg> from <a> to <b>` whose body has the changelog highlights, advisory id if any, and the test evidence.
   - Red: revert, record the failure and the failing test in the ledger, defer the package.
7. Update the ledger and end the iteration. One package per iteration, always.

## Exit Conditions

Scheduled patrols (cron or a long interval) idle when there is nothing to do. Stop conditions (checked at step 2):

| Condition | Action |
|-----------|--------|
| Open patrol PRs exceed cap (default 3) | Pause: merging has fallen behind opening |
| Critical advisory with no fixed version | Report loudly, suggest mitigations |
| Lockfile churn without version changes | Stop: something is wrong with install |

## Interval

Daily or weekly. Use a real scheduler (`/schedule`, cron) rather than a tight interval loop; dependency drift moves slowly.

## Common Mistakes

- Batching ten bumps into one PR: when tests fail nobody knows which bump did it, and the whole PR stalls.
- Trusting the version number instead of the changelog: patch releases ship behavior changes; skim the release notes into the PR body.
- Auto-merging green PRs: green means safe to review, not safe to ship; merge authority stays human unless explicitly granted.
- Ignoring transitive advisories because no direct dep changed: audit output, not the diff, is the source of truth.

## Example

```
/schedule weekly Patrol dependencies in adamcwade/invoiceflow using
patrolling-dependencies. PR cap 3, majors report-only.
```

Ledger entry the loop should produce:

```
[patrol 2026-06-08] audit clean. Bumped prisma 6.2.1 -> 6.2.3 (patch).
Suite 14/14 green, build ok. Opened PR #31 with changelog notes.
zod major 4.x available: report-only, breaking changes in error maps.
```
