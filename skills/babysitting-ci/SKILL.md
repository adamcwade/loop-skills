---
name: babysitting-ci
description: Use when a pull request or branch has CI checks that need watching until green, when checks fail and need diagnosis and fixes, or when you would otherwise poll a checks page by hand waiting for a build, lint, or test job to finish.
---

# Babysitting CI

## Overview

Watch a PR's checks on a loop. Each iteration either confirms progress, diagnoses a failure and pushes a fix, or exits. The loop replaces the human habit of refreshing the checks tab.

Core principle: one iteration handles at most one failure, and every iteration starts by checking the exit conditions and ends by writing the ledger, so the next iteration never repeats work.

## When to Use

- A PR is open and you want it green without watching it
- Checks fail intermittently (flaky tests, transient infra) and need retry-or-fix judgment
- A long build pipeline gates a merge you care about

When NOT to use: checks that need human secrets or approvals to rerun, or a PR you do not have push access to. Report instead of looping.

## The Ledger

All state lives in `.loop/ci-babysit.md`, called the ledger. Every entry records: timestamp, PR number, commit watched, status of each check, the failure handled this iteration (check name plus test name if applicable), what was done and why, and running counters: fix attempts per failure, flake reruns per check, total fix attempts.

Iteration 1 bootstrap: if the ledger does not exist, create `.loop/` and the file, write a baseline entry (PR, commit, current check status, all counters zero), then continue.

## One Iteration

1. Read the ledger (bootstrap it if missing).
2. Fetch check status: `gh pr checks <pr> --json name,state,link`.
3. Check EVERY exit condition in the table below against the fresh status and the ledger counters. Any row matches: do its action and stop. This happens before any fixing.
4. All green: write the final ledger entry, summarize, EXIT.
5. Checks still running and nothing failed: write a status entry, end the iteration, and wait for the loop's next scheduled run.
6. Something failed: pick the FIRST failed check in the list (deterministic order beats clever prioritization). Get its log: follow the check's `link` to the run, or `gh run list --branch <branch> --json databaseId,name` to find the run id, then `gh run view <id> --log-failed`. Classify:
   - Real defect: reproduce locally when a single command can do it, fix, commit, push. Increment that failure's fix-attempt counter and the total.
   - Flake (timeout, network blip, or a test the ledger already marked flaky): rerun with `gh run rerun <id> --failed`. Increment that check's flake counter. Reruns do not consume fix attempts.
   - Infra or permissions (e.g. "Resource not accessible by integration"): stop the loop and report; this needs a human.
7. Write the ledger entry.

## Exit Conditions (checked at step 3, every iteration)

| Condition | Action |
|-----------|--------|
| All checks green | Summarize and stop |
| Same failure (check plus test name) fixed twice and failed again | Stop, report: fix is not working. List any other failures still open |
| Same check rerun as flake 3 times | Stop, report: not a flake |
| Total fix attempts reached the budget (default 3) | Stop, report remaining failures |
| New commits not pushed by this loop (compare ledger's own pushes) | Stop, report: ownership unclear |

Any stop-and-report lists ALL currently failing checks, not just the one being worked.

## Interval

5 to 10 minutes suits most pipelines. Match the slowest job: a 40 minute build does not need a 2 minute loop.

## Common Mistakes

- Fixing several failures in one iteration: each push restarts CI, so later fixes are guesses against stale logs. One failure per iteration.
- Rerunning a real failure as a flake: classify from the log first; rerunning hides defects.
- Fixing before checking exit conditions: a third attempt at a twice-failed fix is the table's job to prevent, but only if it is read first.
- Letting counters live in your head: every counter the exit table needs must be in the ledger.

## Example

```
/loop 8m Babysit CI on PR #142 using the babysitting-ci skill. Budget: 3 fix attempts.
```

Ledger entry the loop should produce:

```
[14:32] PR #142, commit 9f3ab21. lint: pass, build: pass, test: FAIL.
Handling: test / test_invoice_rounding. Log shows assertion off by 0.01.
Real defect, not flake. Fixed banker's rounding in totals.ts, pushed 4c11d02.
Counters: this failure 1/2, flake reruns 0/3, total attempts 1/3.
```
