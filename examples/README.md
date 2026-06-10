# Examples

Copy-paste invocations for each skill, with the kind of ledger output a healthy loop produces.

## babysitting-ci

```
/loop 8m Babysit CI on PR #142 using the babysitting-ci skill. Budget: 3 fix attempts.
```

Healthy ledger (`.loop/ci-babysit.md`):

```
[14:24] PR #142, commit 9f3ab21. lint: pass, build: running, test: running. Waiting.
[14:32] test: FAIL. test_invoice_rounding off by 0.01. Real defect, not flake.
        Fixed banker's rounding in totals.ts, pushed 4c11d02. Attempt 1 of 3.
[14:41] All checks green on 4c11d02. Exiting: success.
```

## grinding-tests-to-green

```
/loop Grind the test suite to green using grinding-tests-to-green. Budget: 20 iterations.
```

Healthy ledger (`.loop/grind.md`):

```
[iter 6] 13 -> 12 failing. Fixed test_tax_rate_lookup: region table renamed.
[iter 7] 12 -> 11 failing. Fixed test_session_expiry: cache key missing tenant id.
         Assertion untouched. Full suite: no new failures. Committed.
[iter 8] 11 -> 11 failing. STALL 1 of 2: test_pdf_render needs a font fixture
         that does not exist in CI. Noted, will attempt fixture next iteration.
```

## watching-deploys

First write the contract, then start the loop:

```
/loop 3m Watch the production deploy that just shipped using watching-deploys.
Thresholds, baseline, and rollback authority are in .loop/deploy-watch.md.
```

Example contract (`.loop/deploy-watch.md`):

```
deploy: invoiceflow v1.4.2, shipped 14:55
checks: GET /api/health expects 200 under 500ms; 5xx rate via vercel logs
thresholds: 5xx above 1% over 5m is unhealthy; p99 above 2x baseline (390ms)
baseline: 5xx 0.2%, p99 390ms (measured 14:30-14:55)
duration: 90 minutes
on failure: page Adam, do NOT roll back without confirmation
```

## triaging-issue-inbox

```
/loop 20m Triage new issues in adamcwade/jobq using triaging-issue-inbox.
Cap 10 items per sweep, never close anything.
```

## patrolling-dependencies

```
/schedule weekly Patrol dependencies in adamcwade/invoiceflow using
patrolling-dependencies. PR cap 3, majors report-only.
```

PR body the patrol should produce:

```
chore(deps): bump prisma from 6.2.1 to 6.2.3

Changelog highlights: fixes connection pool leak under PgBouncer, no API changes.
Advisory: none. Tests: 14/14 green. Build: ok.
Opened by the patrolling-dependencies loop; merge stays with humans.
```
