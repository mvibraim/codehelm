# CodeHelm

<p align="center">
  <img
    src="plugins/codehelm/.claude-plugin/icon.png"
    alt="CodeHelm"
    width="300"
  />
</p>

<p align="center">
  <strong>Let Claude write the code. You steer the ship.</strong>
</p>

CodeHelm is a Claude Code plugin that takes a change from brief to delivery in four skills (specify, plan, implement, audit), stops for your approval at each gate and ships one tested task per pull request.

## Install

In a Claude Code session (v2.1.275 or later), run:

```text
/plugin install codehelm --marketplace mvibraim/codehelm
```

On older versions, run `/plugin marketplace add mvibraim/codehelm`, then `/plugin install codehelm@codehelm`.

To update, run `claude plugin update codehelm@codehelm`, or turn on auto-update for the `codehelm` marketplace in `/plugin`.

## Quick start

```text
/model opus
/effort high
/codehelm:plan-tasks Add rate limiting to the public API
  CodeHelm  3 questions: limit per key or per IP? 429 body? store?
  you       1 per key, 2 JSON error, 3 in-memory
  CodeHelm  plan: S01 per-key limit, S02 JSON 429; tests npm test, npm run test:int; ADR-0004
  you       approve
  CodeHelm  wrote docs/plans/rate-limiting/plan.md and docs/adr/0004-in-memory-limiter.md
you: commit on docs/plan-rate-limiting, merge its pull request, start feat/rate-limiting-per-key
/clear
/model sonnet
/effort medium
/codehelm:implement rate-limiting
  CodeHelm  S01: baseline green, 4 new tests fail, implemented, all checks pass
  you       accepted
  CodeHelm  git add src/limiter.ts test/limiter.test.ts docs/plans/rate-limiting/plan.md
            git commit -m 'feat: add per-key rate limiter' -m 'Task: rate-limiting/S01'
you: run them, merge S01's pull request, start feat/rate-limiting-json-429
/clear
/codehelm:implement rate-limiting      # S02, same loop, you commit
/clear
/codehelm:audit rate-limiting          # runs on Opus at high by itself
  CodeHelm  9 VERIFIED, 0 findings: ready for delivery
you: commit verification.md, open S02's pull request
```

Each handoff ends with a **Next** block that spells out your steps. Skills hand off through files, never chat, so `/clear` loses nothing, but don't clear while a task waits for your review.

## The four skills

| Command | Writes | Stops for you |
| --- | --- | --- |
| `/codehelm:specify <brief>` (optional) | `docs/specs/<spec-slug>.md`, one business feature | questions, then approval |
| `/codehelm:plan-tasks <request, spec or plan>` | `docs/plans/<plan-name>/plan.md` and an ADR in `docs/adr/` per lasting decision | questions, then approval |
| `/codehelm:implement <plan> [task]` | tests, then code for one task | review, until you reply "accepted"; earlier only when something is off |
| `/codehelm:audit [plan] [task]` | `verification.md` | never |

Use Opus at `high` effort for `/codehelm:specify` and `/codehelm:plan-tasks`, and Sonnet at `medium` for `/codehelm:implement`; each skill notes a mismatch in one line. `/codehelm:audit` runs on Opus at `high` by itself, in a fresh subagent.

## Rules

1. **Nothing happens without your approval.** Specs and plans are saved only once you approve them, and a source that changes after approval stops the work.
2. **You own Git.** CodeHelm only suggests branches, commits and pull requests; you run them. Every skill except `/codehelm:specify` needs a Git repository.
3. **One task per run and per pull request.** A task goes from `NOT STARTED` to `READY FOR HUMAN REVIEW` (every check passed) to `DONE` (you accepted it), or to `BLOCKED — <reason>`. `/codehelm:implement` takes the task on the current branch, or the next one whose dependencies are `DONE`.
4. **Tests gate every task.** The plan's test commands are approved with the plan and run before editing (except what a setup task creates and a resumed task's own tests) and after. A failure the task didn't cause blocks it.
5. **Evidence, not claims.** Only `/codehelm:audit` marks a requirement `VERIFIED`, never takes "tests passed" as proof and never changes code or task status.

## Git flow

One feature takes, in order:

1. **Spec** (optional): `docs/specify-<spec-slug>`, one commit, one pull request. Merge it before you approve the plan.
2. **Plan**: `docs/plan-<plan-name>`, with `plan.md` and its new ADRs. Merge it before the first task. A plan with no new ADR and at most two tasks commits on the first task's branch instead.
3. **Tasks**: one branch per task, named in the plan, each with one commit and one pull request. Reply "accepted" before you commit, so `DONE` lands in that commit. Start each branch from the default branch once its dependencies merge; independent tasks can run in parallel.
4. **Audit**: the full audit's `verification.md` ships in the last task's pull request, or on `docs/audit-<plan-name>` when run from the default branch.

## License

CodeHelm is released under the [MIT License](LICENSE).
