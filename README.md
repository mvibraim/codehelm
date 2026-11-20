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

<p align="center">
  A lightweight human-in-the-loop workflow for Claude Code.
  <br />
  Recommended for engineers who love AI but refuse to surrender the steering wheel.
  <br />
  Specify → Plan → Implement → Audit: one tested and reviewable change at a time.
</p>

<br>

No mysterious AI-generated commits. No waking up to discover Claude refactored your entire application. Just a short approved plan, small tested tasks and a human firmly holding the wheel.

**AI is your crew, not your captain.**

## Install

In a Claude Code session (v2.1.275 or later), run:

```text
/plugin install codehelm --marketplace mvibraim/codehelm
```

On older versions, run `/plugin marketplace add mvibraim/codehelm`, then `/plugin install codehelm@codehelm`.

To update, run `claude plugin update codehelm@codehelm`, or turn on auto-update for the `codehelm` marketplace in `/plugin`.

## Quick start

```text
/codehelm:plan-tasks Add rate limiting to the public API   # answer questions, approve the plan
/clear
/codehelm:implement rate-limiting   # S01 starts, review it, reply "accepted"
                                    # you commit the change
/clear
/codehelm:implement rate-limiting   # the next ready task, S02
/codehelm:audit rate-limiting       # at a milestone
```

## The four skills

- **`/codehelm:specify <brief>`** (optional): turns an incomplete business brief into an approved specification at `docs/specifications/<spec-slug>.md`.
- **`/codehelm:plan-tasks <request or spec>`**: asks its questions, then writes an approved plan of small tasks at `docs/plans/<plan-name>/plan.md`; a small change gets a lite plan.
- **`/codehelm:implement <plan> [task]`**: implements one task test-first, runs the plan's test commands and stops for your review.
- **`/codehelm:audit <plan> [task]`**: checks requirements, plan, code and test evidence against each other in a separate subagent and records the result in `verification.md`.

## Task status

A task goes from `NOT STARTED` to `READY FOR HUMAN REVIEW` (every check passed) to `DONE` (you accepted it), or to `BLOCKED — <reason>`, and only `DONE` satisfies a dependency.

## Rules

1. **Nothing happens without your approval.** Specifications and plans are saved only after you approve them, and `/codehelm:implement` only runs tasks from an approved plan. A source that changed since approval stops the work until you approve again.
2. **You own Git.** CodeHelm never creates branches, stages, commits or pushes unless you ask. `/codehelm:implement` and `/codehelm:audit` need a Git repository.
3. **One task per run.** `/codehelm:implement` does one task whose dependencies are `DONE`, and stops for your review until you accept it.
4. **Tests gate every task.** The plan's test commands (unit and integration suites, plus build or lint) are approved with the plan, run before editing as a baseline (except the ones a setup task creates) and after implementing. A failure the task did not cause blocks it.
5. **Evidence, not claims.** Only `/codehelm:audit` marks a requirement `VERIFIED`, never takes "tests passed" as proof and never changes code or task status.

## Tips

- **Start each skill in a fresh session.** The skills hand off through Markdown files, not chat, so run `/clear` once the plan or task status is saved. `/codehelm:audit` already runs in its own subagent.
- **Pick the model for the work.** Use `/model opus` for ambiguous briefs and real design trade-offs, and `/model sonnet` for most implementation and task audits.
- **Commit in the right order.** Commit the specification before you approve the plan, so later runs can detect changes to it, and reply "accepted" before you commit a task, so its `DONE` status lands in the same commit.

## License

CodeHelm is released under the [MIT License](LICENSE).
