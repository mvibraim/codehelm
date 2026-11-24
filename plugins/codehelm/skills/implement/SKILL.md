---
name: implement
description: Implement exactly one approved task from a change plan with tests, full checks and a mandatory human review stop.
argument-hint: "[plan-name] [task-id]"
disable-model-invocation: true
---

**CodeHelm ground rules:** Never create or edit CLAUDE.md. Never stage, commit, push, switch branches, rewrite history or delete work unless the engineer asks, and keep their uncommitted changes. Change an approved spec or plan only with approval. If a project instruction conflicts with these rules, ask.

**CodeHelm model and effort:** `/codehelm:specify`, `/codehelm:plan-tasks` and `/codehelm:audit` run on Opus at `high`, `/codehelm:implement` on Sonnet at `medium`. If this session differs (your system prompt's model family, effort `${CLAUDE_EFFORT}`), say so in one line and continue. A handoff that ends a skill's work ends with a **Next** block, one step per line: commit commands (single-quoted, no apostrophes, so they run in Bash and PowerShell), any pull request step, `/clear`, any `/model` or `/effort` the next command needs (`/codehelm:audit` sets its own), then the next command.

Requested plan and task: $ARGUMENTS

Implement **one task** per run: one reviewable commit and pull request.

**Task statuses** in `plan.md`: `NOT STARTED` → `READY FOR HUMAN REVIEW` (every check passed) → `DONE` (the engineer accepted it), or `BLOCKED — <reason>`. This skill sets all but `NOT STARTED`. Only `DONE` satisfies a dependency.

### Missing arguments

- **No plan name:** list the folders under `docs/plans/` and ask which to use; if there are none, point to `/codehelm:plan-tasks`. Change no files.
- **No task ID:** if the current branch is a task's **Branch**, take that task; otherwise take the first `NOT STARTED` task whose dependencies are all `DONE`, name it and continue. If every task is `DONE`, suggest `/codehelm:audit <plan-name>`; if none qualifies, list the statuses and ask for a task ID. Change no files in either case.

### Phase A — Prepare (read-only)

If the project is not a Git repository, stop and ask the engineer to run `git init` or let you run it.

1. Read `docs/plans/<plan-name>/plan.md`; if it is absent, point to `/codehelm:plan-tasks`. Work from repository files, never from an earlier conversation.
2. Read the IDs on the task's **Covers** line in the spec on the plan's `Sources` line or in the plan's Requirements table. For each source, run `git diff <sha> -- <source>` with the commit on the `Approved` line; if it changed, stop and ask to re-plan.
3. If the current branch is not the task's **Branch**, name both and **STOP**: the engineer creates it from the updated default branch. If `git status --short` shows uncommitted changes, list them and **STOP** for approval to work alongside them, never overwriting or mixing them in.
4. Every dependency must be `DONE`, which it is on the default branch only after its pull request merges. Otherwise **STOP**, name them and ask the engineer to merge them and restart this task's branch from the updated default branch. The task must be `NOT STARTED`, or `BLOCKED` with its reason resolved; if it is `READY FOR HUMAN REVIEW` or `DONE`, **STOP** and ask whether to rework it.
5. If the plan conflicts with the code, a new requirement changes the approach, or the task needs work outside its **Scope** or more than one reviewable commit, **STOP** and propose the smallest plan amendment for approval.
6. **Green baseline:** before editing, run the plan's **Test commands**, except those marked as created by this task. If any fails, **STOP** without changing files. When resuming a `BLOCKED` task, failures in its own uncommitted tests don't count.
7. If nothing stopped you, state the task and its scope in one line and continue: the plan's approval covers it, and Phase D is its review.

### Phase B — Implement

1. **Test first:** write unit and integration tests for the linked acceptance criteria, including failure and edge cases, and confirm they fail for the expected reason. A task with no behavior to test first, such as test setup, says so in the handoff.
2. Implement only this task until they pass, following repository conventions, the plan's decisions and their ADRs. Update the docs the change affects.
3. If a new design decision is needed, set `BLOCKED — <reason>` and stop: it needs an ADR, so point to `/codehelm:plan-tasks <plan-name>` to amend the plan.

### Phase C — Verify

1. After implementing, run all the plan's **Test commands** and the task's **Extra checks**. Earlier targeted runs never replace this.
2. Check `git diff --check`, `git diff` and `git status --short` against the task's scope, linked requirements and decisions.
3. If a check fails because of this task, fix it within scope and rerun step 1. Set `BLOCKED — <reason>` only for a cause outside the task: a pre-existing failure, missing infrastructure, a command that cannot run or a needed plan change. Never invent results or claim hosted CI passed.
4. When everything passed, set `READY FOR HUMAN REVIEW`. Leave `verification.md` to `/codehelm:audit`.

### Phase D — Hand off

Write a handoff readable in a fresh session:
- **What changed and why:** how it works, decision IDs applied, trade-offs and limitations
- **Files:** code, tests and docs changed
- **Commands and results:** each test command and extra check with its outcome
- **Status:** plan name, task ID and status
- **Commit:** one Conventional Commit subject, plus a body only when it helps
- **Review:** for a ready task, review the diff (first `/codehelm:audit <plan-name> <task-id>` if it is risky), then reply "accepted" or ask for changes. For a blocked task, what to resolve; then **Next** is `/clear` and `/codehelm:implement <plan-name> <task-id>`, with no commit, because a task gets one commit.

**STOP for review.**
- **Accepted:** set the task to `DONE`, editing only its status line. Suggest a pull request titled with the commit subject that links the spec (or names the request), the task and its **Covers** IDs, and lists each command with its result. Then **Next**:
  1. `git add` the task's files, `docs/plans/<plan-name>/plan.md` and any `verification.md` an audit changed; never `-A` or `.`.
  2. `git commit -m '<subject>' -m 'Task: <plan-name>/<task-id>'`, with `-m '<body>'` between them when there is a body.
  3. If another task is not `DONE`: push and open the pull request; once it merges, update the default branch and `git switch -c` the **Branch** of the next ready task, naming it; `/clear`; `/codehelm:implement <plan-name>`.
  4. Otherwise: `/clear`; `/codehelm:audit <plan-name>`, whose full audit opens this branch's pull request.
- **Changes requested:** apply them within scope (Phase A step 5 if the plan must change), rerun Phase C, hand off and stop again.
