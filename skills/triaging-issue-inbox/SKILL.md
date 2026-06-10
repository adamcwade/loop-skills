---
name: triaging-issue-inbox
description: Use when new GitHub issues or pull requests arrive faster than they get labeled and answered, when a repo inbox needs regular triage (labels, duplicates, missing repro info), or when maintainers want first-response time kept low without watching notifications.
---

# Triaging an Issue Inbox

## Overview

Sweep a repository's new issues and PRs on a loop: label, detect duplicates, request missing information, and flag the items that genuinely need a maintainer. The loop keeps first-response time low; it does not make maintainer decisions.

Core principle: the loop drafts and organizes, a human decides. Closing, merging, and committing to fixes stay with maintainers unless explicitly pre-authorized.

## When to Use

- A public repo gets steady issue traffic and triage lags
- After a release or launch spike floods the inbox
- You maintain several repos and want one loop sweeping all of them

When NOT to use: repos where you lack triage permission (the loop can only draft, which is fine, but say so), or communities with a triage rota and conventions the loop has not been given.

## One Iteration

1. Read the ledger, `.loop/triage.md`. If it does not exist, create `.loop/` and the file with a baseline entry: the repos to sweep, the current newest issue number per repo as the starting high-water mark, and counters at zero. Triage starts with items newer than the baseline.
2. Check the stop conditions table below against the ledger. Any row matches: do its action.
3. Fetch only newer items: `gh issue list --state open --json number,title,body,labels,createdAt` filtered past the high-water mark. Nothing new: write state, end iteration.
4. For each new item, up to the per-iteration cap (default 10):
   - Classify: bug, feature request, question, or docs. Apply labels.
   - Duplicate check: search existing issues for matching titles and error strings. Likely dup: comment linking the original with "possible duplicate" phrasing, never close.
   - Bug without repro steps, version, or logs: post the repo's info-request template.
   - Security report, data loss, or angry-customer tone: add the escalation label and put it in the maintainer digest with a one-line reason.
5. Append to the ledger: items handled, labels applied, escalations. Advance the high-water mark.
6. End of a sweep day (or every N iterations): post or save a digest: counts, escalations, oldest unanswered item.

## Exit Conditions

This loop is continuous; it idles on empty iterations rather than exiting. Stop conditions (checked at step 2):

| Condition | Action |
|-----------|--------|
| Rate limit hit | Note reset time, end iteration early |
| Same item reappears with comments | Leave it alone, humans are talking |
| Asked to close or argue with users | Out of scope, flag to maintainer |

## Interval

15 to 30 minutes for active repos; an hour or more for quiet ones. Faster polling than your issue arrival rate just burns rate limit.

## Common Mistakes

- Closing duplicates instead of linking them: false-positive closes cost community trust that labels cannot buy back.
- Re-triaging the whole inbox each iteration: the high-water mark exists so old threads are never touched twice.
- Templated replies that ignore the issue text: quote the user's actual error line when asking for more info.
- Burying escalations in labels only: security and data-loss reports go in the digest with a reason, loudly.

## Example

```
/loop 20m Triage new issues in adamcwade/jobq and adamcwade/envcheck using
triaging-issue-inbox. Cap 10 items per sweep, never close anything.
```

Ledger entry the loop should produce:

```
[sweep 9] 3 new. #87 labeled bug, asked for Node version (no repro).
#88 likely dup of #71, linked. #89 labeled question, drafted answer for review.
0 escalations. High-water mark now #89.
```
