---
name: grinding-tests-to-green
description: Use when a test suite has multiple failures to burn down, after a large refactor or dependency upgrade breaks many tests, or when a backlog of TODO or FIXME items should be worked through one item at a time until none remain.
---

# Grinding Tests to Green

## Overview

Burn down a pile of failures one fix per iteration until the suite is green. The loop turns an overwhelming wall of red into a sequence of small, verified fixes.

Core principle: fix exactly one failure per iteration, verify it, commit it. Small verified steps compound; big speculative ones regress.

## When to Use

- A refactor or upgrade broke dozens of tests
- Flaky-free suite where every failure is a real defect to chase
- Any countable backlog: lint errors, type errors, deprecation warnings, FIXMEs

When NOT to use: failures that share one root cause (fix the cause once instead, then rerun), or a suite so flaky that green is not a stable target (fix flakiness first).

## The Ledger

All state lives in `.loop/grind.md`, called the ledger. Every entry records: iteration number, count before, count after, the exact counting command used, what was fixed and how (code fix or assertion change, with justification), and the stall counter.

Pick ONE counting command up front (`npm test`, `tsc --noEmit`, `grep -rc FIXME src`) and record it in the ledger baseline. "The count" always means that command's full, unfiltered output; piping through `tail` is for reading, never for counting.

Iteration 1 bootstrap: if the ledger does not exist, create `.loop/` and the file, run the counting command, and write a baseline entry (count, command, stall counter 0). That is the whole first iteration, always; fixing starts at iteration 2.

## One Iteration

1. Read the ledger (bootstrap it if missing).
2. Run the counting command and get the current count.
3. Check EVERY exit condition in the table below before touching code. Any row matches: do its action and stop.
4. Pick the FIRST failure in the output (deterministic order beats clever prioritization). Read the failing test and the code under test. Fix the defect, not the assertion, unless the test itself is provably wrong; the ledger entry must say which and why.
5. Rerun just that test, then run the full counting command again. Commit with the failure name in the message.
6. Write the ledger entry: count before, count after, what was fixed, stall counter (reset to 0 if the count dropped, incremented if it did not).

## Exit Conditions (checked at step 3, every iteration)

| Condition | Action |
|-----------|--------|
| Count is zero | Run the counting command once more to confirm, summarize all fixes, stop |
| Count went UP versus the ledger | Revert the last commit (`git revert HEAD`), confirm the count returned to its prior value, then stop and report which fix regressed |
| Stall counter at 2 (count flat two iterations) | Stop, report what is stuck |
| Budget reached (iterations or tokens) | Stop, report remaining count |

## Interval

Self-paced (`/loop` with no interval) is ideal: each iteration starts when the previous fix lands. Use a timed interval only when the suite is slow.

## Common Mistakes

- Batch-fixing five tests at once: when the suite then fails, you cannot tell which fix regressed it.
- Editing assertions to match broken behavior: the ledger must justify every assertion change.
- Skipping the full count after a fix: a fix that breaks two other tests looks like progress until iteration end.
- Counting from truncated output: stall detection and the count-went-up exit both depend on the real number.
- No committed checkpoint per fix: per-fix commits are what make the revert-on-regression exit possible.

## Example

```
/loop Grind the test suite to green using grinding-tests-to-green. Budget: 20 iterations.
```

Ledger entry the loop should produce:

```
[iter 7] 12 -> 11 failing (npm test). Fixed test_session_expiry: cache key
missing tenant id after refactor. Code fix in session.ts, assertion untouched.
Full count rerun, no new failures. Stall 0. Committed "fix tenant scoping".
```
