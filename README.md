# loop-skills

Five Claude Code skills for running agent loops well.

An agent loop is a prompt that runs repeatedly, on a timer or self-paced, until a condition is met: "check CI every 8 minutes and fix what breaks", "fix one failing test per iteration until the suite is green". Loops are easy to start and easy to run badly. A bad loop repeats work, fixes five things at once, forgets what it already tried, and never knows when to stop.

These skills encode the discipline that makes loops work. Each one covers a high-value use case and gives the agent an iteration contract: what one iteration does, what state it persists, and exactly when to exit.

## The skills

| Skill | Use it when | Loop style |
|-------|-------------|------------|
| [babysitting-ci](skills/babysitting-ci/SKILL.md) | A PR needs to get green without you watching the checks tab | Timed, exits on green |
| [grinding-tests-to-green](skills/grinding-tests-to-green/SKILL.md) | A refactor broke dozens of tests, or any countable backlog needs burning down | Self-paced, exits at zero |
| [watching-deploys](skills/watching-deploys/SKILL.md) | A release just shipped and needs monitoring with a promote-or-rollback verdict | Timed, fixed duration |
| [triaging-issue-inbox](skills/triaging-issue-inbox/SKILL.md) | New issues arrive faster than they get labeled and answered | Timed, continuous |
| [patrolling-dependencies](skills/patrolling-dependencies/SKILL.md) | Updates and security advisories should arrive as small tested PRs on a schedule | Scheduled, continuous |

## Install

As a Claude Code plugin:

```
/plugin marketplace add adamcwade/loop-skills
/plugin install loop-skills@loop-skills-marketplace
```

Or copy any skill directly into your personal skills directory:

```bash
git clone https://github.com/adamcwade/loop-skills
cp -r loop-skills/skills/babysitting-ci ~/.claude/skills/
```

## Usage

Start a loop and name the skill. With Claude Code's `/loop`:

```
/loop 8m Babysit CI on PR #142 using the babysitting-ci skill. Budget: 3 fix attempts.
```

Self-paced (the model decides when to run the next iteration):

```
/loop Grind the test suite to green using grinding-tests-to-green. Budget: 20 iterations.
```

For slow-moving work like dependency patrol, prefer a real schedule over a tight loop:

```
/schedule weekly Patrol dependencies in my-org/my-repo using patrolling-dependencies.
```

More copy-paste invocations are in [examples/](examples/).

## The five rules behind every skill

The skills share one design philosophy. If you write your own loop skills, these transfer:

1. One unit of work per iteration. One failure fixed, one package bumped, one sweep of new items. Batching destroys the feedback signal that makes loops safe.
2. Persist a ledger. Every iteration reads state from a file (`.loop/*.md`) and appends what it did. A loop without memory retries failed strategies forever.
3. Written exit conditions. Success, stall, regression, and budget exhaustion are all defined before iteration one. "I'll know it when I see it" is how loops run all night.
4. Escalate instead of improvise. The loop's authority is whatever was agreed up front. Anything outside it gets reported to a human, loudly, and the loop stops.
5. Match the interval to the signal. Poll a 40 minute build every 8 minutes, not every 2. Check dependencies weekly, not hourly. Faster than the underlying signal is pure waste.

## Repository layout

```
loop-skills/
  .claude-plugin/        plugin and marketplace manifests
  skills/
    babysitting-ci/SKILL.md
    grinding-tests-to-green/SKILL.md
    watching-deploys/SKILL.md
    triaging-issue-inbox/SKILL.md
    patrolling-dependencies/SKILL.md
  examples/              copy-paste loop invocations and sample ledgers
```

## Writing your own loop skill

Start from the closest existing skill and keep its skeleton: Overview with one core principle, When to Use (including when NOT to), One Iteration as a numbered list, an exit conditions table, interval guidance, common mistakes, and one realistic example with a sample ledger entry. The skeleton is the value; the use case is the variable.

## License

MIT, see [LICENSE](LICENSE).
