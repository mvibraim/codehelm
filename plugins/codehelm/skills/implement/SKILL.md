---
name: implement
description: Implement exactly one approved task from a change plan with tests, full checks and a mandatory human review stop.
argument-hint: "[plan-name] [task-id] [--done]"
disable-model-invocation: true
---

**CodeHelm ground rules:** Follow the project's CLAUDE.md and conventions, but never create or edit CLAUDE.md. Never stage, commit, push, switch branches, rewrite history or delete work unless the engineer asks for that action. Keep the engineer's uncommitted changes intact. Change an approved spec or plan only with approval. If a project instruction conflicts with these rules, ask.

Requested plan and task: $ARGUMENTS

**One invocation = one task = one proposed reviewable commit.** Do not implement a second task or automatically commit.

Task statuses (`NOT STARTED`, `BLOCKED — <reason>`, `READY FOR HUMAN REVIEW`, `DONE`) follow the task status lifecycle section in `plan.md`.

### Missing arguments (read-only)

Apply this check first, with or without `--done`. Never guess a plan or task.

- **No plan name:** list the folders under `docs/plans/` and ask which plan to use. If there are none, report that and direct the engineer to `/codehelm:plan-tasks`. **STOP.**
- **Plan name but no task ID:** read that plan's `plan.md` and list its tasks with their statuses (for `--done`, point out the `READY FOR HUMAN REVIEW` tasks). Ask for the exact task ID. **STOP.**

Change no files while arguments are missing; continue only after the engineer gives both values.

### Mark a task DONE (`--done`)

`/codehelm:implement <plan-name> <task-id> --done` records that the engineer has reviewed and accepted the task. When `--done` is present, follow only these steps and skip Phases A–D:

1. Apply "Missing arguments" above. Read `docs/plans/<plan-name>/plan.md` and confirm the task exists. If the plan or task is missing, stop and report it.
2. If the task is `READY FOR HUMAN REVIEW`, the invocation itself is the approval: set it to `DONE`. If it is `NOT STARTED` or `BLOCKED — <reason>`, show the current status, warn that it has not passed `/codehelm:implement` verification and set `DONE` only after explicit confirmation. If it is already `DONE`, report that and change nothing.
3. Edit only that task's status line. Do not change code, tests, `verification.md` or any other task.
4. Report the old and new status, then remind the engineer to include `plan.md` in the task's commit. **STOP.**

### Phase A — Prepare (read-only, except confirmed DONE statuses)

If the project is not a Git repository, stop and ask the engineer to run `git init` or to approve that you run it.

1. Parse `<plan-name> <task-id>`; if either is missing, follow "Missing arguments" above. If `--done` is present, follow "Mark a task DONE" above instead. Read `docs/plans/<plan-name>/plan.md` and `decisions.md`. If the plan artifacts are absent, stop and direct the engineer to `/codehelm:plan-tasks` rather than inventing them. **Treat repository artifacts as authoritative; never depend on a previous conversation to know the task.**
2. Confirm the task exists and has been approved; review only linked acceptance criteria and relevant sections in original specifications. Check source version/hash if recorded. If a source changed, stop and ask to re-plan. Use targeted file search; do not dump the entire repo, unrelated history, generated files or long logs into context.
3. Inspect the repository, the plan's test commands, `git status --short` and the current branch. Do not overwrite/mix preexisting uncommitted work or silently switch/create a branch.
4. Read task statuses in `plan.md`:
   - If any other task is still `READY FOR HUMAN REVIEW`, ask the engineer in one question which of those tasks they have reviewed and accepted. Immediately set each confirmed task to `DONE`: the answer is the approval for those status lines. This is the only edit allowed in Phase A. Leave unconfirmed tasks unchanged.
   - Every dependency of the selected task must be `DONE`. Otherwise **STOP** and name the unfinished dependencies.
   - The selected task must be `NOT STARTED` or `BLOCKED — <reason>`; resume a blocked task only when its reason is resolved. If it is `READY FOR HUMAN REVIEW` or `DONE`, **STOP** and ask whether to rework it.
5. If plan/decisions/code conflict, a newly discovered requirement changes the approach or the task is too large for one reviewable commit, **STOP**. Explain the smallest required plan/decision amendment and request approval; never silently redesign or expand scope.
6. Present a **short execution proposal**: exact scope, likely files, relevant tests/specialized checks, expected doc updates and risks.
7. If step 4 marked earlier tasks `DONE`, check `git diff` for `plan.md`. When those status lines are its only uncommitted changes, recommend committing `plan.md` on its own before approving the proposal (for example, `docs: mark S02 as done`), so the next task's diff holds only its own work. This is optional; do not stage or commit it yourself. If `plan.md` has other uncommitted changes, do not recommend the separate commit; say that the status changes will be part of this task's diff instead.
8. **STOP for explicit approval before editing files.**

### Phase B — Implement (only after approval)

1. Implement only the selected task, following repository conventions and approved decisions.
2. Add/update meaningful unit and integration tests for the behavior. Test failure cases and edge conditions relevant to this task. Avoid unrelated refactors and unnecessary dependencies.
3. Keep docs aligned with code, including API/configuration/usage examples and accepted changes. If new design decisions become necessary, set the task to `BLOCKED — <reason>` and stop for approval before deviating from the plan.
4. Preserve all unrelated user changes.

### Phase C — Mandatory verification

1. **After implementation is complete**, run every command in the plan's **Test commands** section, then the task's **Extra checks**. Execute the project-specific commands directly; CodeHelm provides no wrapper script. You may run targeted tests earlier for feedback, but they never replace these post-implementation full runs. If a command is missing or cannot run, report the blocker and do not set `READY FOR HUMAN REVIEW` (see step 4).
2. Capture concise success summaries and full relevant failure diagnostics; avoid unnecessary large tool output.
3. Inspect `git diff --check`, `git diff` and `git status --short`. Validate the scope and compare implemented behavior and tests against linked requirements and approved `decisions.md`.
4. If any test command or extra check cannot run or fails, report the exact blocker and set the task to `BLOCKED — <reason>`; do **not** set `READY FOR HUMAN REVIEW`. Do not invent results. This skill never pushes: hosted CI success is checked after the engineer's approved push, by `/codehelm:audit` (from the CI run for the audited revision) or by the engineer. A local YAML lint is not equivalent.
5. Only when every test command and extra check passed, set the task to `READY FOR HUMAN REVIEW` (not committed). The status records the result: `READY FOR HUMAN REVIEW` means every check passed, and `BLOCKED — <reason>` says what failed. The commands and their results go in the Phase D handoff. Do not create or edit `verification.md`; only `/codehelm:audit` writes it. Never set `DONE` for the task just implemented; only the engineer accepts it, through `--done` or at the start of a later run.

### Phase D — Hand off

Report concisely:
- What changed, why and how it works (explain important code paths)
- Files changed, key trade-offs and any documentation changes
- Exact test commands and extra checks run, with observed outcomes
- Known limitations and requirements needing audit
- Suggested single Conventional Commit message
- Suggested review commands: `git diff --check`, `git diff`, `git status --short`
- **Fresh-session handoff:** plan name + task ID + approved decisions referenced, task status (`READY FOR HUMAN REVIEW` or `BLOCKED — <reason>`), exact checks and outcomes, pending human action and the next command. If earlier tasks were marked `DONE` in this run and those status changes were not committed separately, say that this diff also contains them. For a ready task, recommend this order: review, optionally `/codehelm:audit <plan-name> <task-id>`, then `/codehelm:implement <plan-name> <task-id> --done`, then commit. Save lasting decisions in Markdown, not only chat.

**STOP.** Wait for the engineer to review, stage and commit manually.
