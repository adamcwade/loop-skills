---
name: watching-deploys
description: Use when a deployment, release, or rollout just shipped and needs monitoring for error spikes, failed health checks, or elevated latency, or when a canary needs watching before promoting to production.
---

# Watching Deploys

## Overview

Monitor a fresh deploy on a loop and escalate the moment evidence says the release is bad. The loop is a smoke detector: its job is fast, accurate escalation, not heroic unsupervised repair.

Core principle: define the healthy/unhealthy thresholds BEFORE the first iteration, in writing. A loop that decides what "bad" means while looking at scary graphs will talk itself into either panic or complacency.

## When to Use

- A production or staging deploy just went out and the team would otherwise watch dashboards
- A canary or rolling release needs a verdict: promote or roll back
- A risky migration shipped and specific symptoms would confirm trouble

When NOT to use: when no logs, metrics, or health endpoints are reachable from the CLI (nothing to observe), or when rollback requires approvals the loop does not have (watch and page instead).

## Setup (before the first iteration)

Write `.loop/deploy-watch.md` containing: deploy id and time, the health checks to run (exact commands or URLs), thresholds (e.g. "5xx rate above 1 percent over 5 min is unhealthy"), baseline numbers from before the deploy, the watch duration, and the agreed action on failure: roll back, or page and stop.

## One Iteration

1. Read the watch file. If it does not exist, do not invent a contract: stop and report that Setup must happen first.
2. Check EVERY exit condition in the table below against the watch file and the last status lines. Any row matches: do its action and stop.
3. Run each health check: hit health endpoints, query error rate and latency (`vercel logs`, `kubectl logs`, curl on /api/health, whatever the file specifies), and compare against thresholds, not vibes.
4. All within thresholds: append one status line to the watch file, end iteration.
5. Threshold breached: confirm it is deploy-correlated (did the metric move at deploy time, or was it already bad in the baseline?). If correlated, execute the agreed action exactly: roll back if pre-authorized, otherwise page the owner and stop. Never improvise a hotfix on production inside a watch loop. Either way, append what was observed and done.

## Exit Conditions (checked at step 2, every iteration)

| Condition | Action |
|-----------|--------|
| Watch duration elapsed, healthy | PASS verdict, summarize, stop |
| Threshold breached, deploy-correlated | Agreed action, then stop |
| Breach not correlated with deploy | Report pre-existing issue, keep watching |
| Observability itself is down | Stop and page: flying blind is a failure |

## Interval

2 to 5 minutes early (most regressions show inside 15 minutes), stretching to 10 to 15 minutes for the remainder. Typical total watch: 1 to 2 hours.

## Common Mistakes

- Thresholds invented mid-loop: the watch file is the contract; if it is wrong, stop and fix it, do not freelance.
- Treating a pre-existing error rate as a deploy regression: always compare against the recorded baseline.
- Hotfixing production from inside the loop: the loop's authority is the agreed action, nothing more.
- Watching only averages: p99 latency and per-route errors break first.

## Example

```
/loop 3m Watch the invoiceflow production deploy that just shipped using
watching-deploys. Thresholds and rollback authority are in .loop/deploy-watch.md.
```

Status line the loop should produce:

```
[15:04] t+12m. /api/health 200 in 84ms. 5xx 0.2% (threshold 1%).
p99 412ms vs baseline 390ms. Healthy, 6 checks remain.
```
