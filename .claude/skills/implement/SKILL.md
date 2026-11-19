---
name: implement
description: Implement exactly one approved task from a change plan with tests, document checks and a mandatory human review stop.
argument-hint: "[plan-name] [task-id] [--done]"
disable-model-invocation: true
---

**CodeHelm portability and control:** This skill is self-contained. Respect an existing project `CLAUDE.md` when present; never create, copy, replace or merge one as a prerequisite. Follow existing repository conventions and preserve the engineer's uncommitted work. Never stage, commit, push, create/switch branches, rewrite history or perform destructive changes without explicit approval for that action. Project specifications and approved decisions are durable sources of truth; do not silently change them. If project instructions conflict with a CodeHelm safeguard, stop and ask the engineer to resolve the conflict. If Git is not initialized, report that fact and never initialize it without approval.

Requested plan and task: $ARGUMENTS

**One invocation = one task = one proposed reviewable commit.** Do not implement a second task or automatically commit.

Task statuses (`NOT STARTED`, `BLOCKED — <reason>`, `READY FOR HUMAN REVIEW`, `DONE`) follow the task status lifecycle section in `plan.md`.

### Mark a task DONE (`--done`)

`/implement <plan-name> <task-id> --done` records that the engineer has reviewed and accepted the task. When `--done` is present, follow only these steps and skip Phases A–D:

1. Read `docs/plans/<plan-name>/plan.md` and confirm the task exists. If the plan or task is missing, stop and report it.
2. If the task is `READY FOR HUMAN REVIEW`, the invocation itself is the approval: set it to `DONE`. If it is `NOT STARTED` or `BLOCKED — <reason>`, show the current status, warn that it has not passed `/implement` verification and set `DONE` only after explicit confirmation. If it is already `DONE`, report that and change nothing.
3. Edit only that task's status line. Do not change code, tests, `verification.md` or any other task.
4. Report the old and new status, then remind the engineer to include `plan.md` in the task's commit. **STOP.**

### Phase A — Prepare (read-only, except confirmed DONE statuses)

1. Parse `<plan-name> <task-id>`; if missing, ask for the exact IDs. If `--done` is present, follow "Mark a task DONE" above instead. Read the project's `CLAUDE.md` **only if it exists**; CodeHelm does not provide one. Read `docs/plans/<plan-name>/plan.md`, `decisions.md` and `verification.md`. If the plan artifacts are absent, stop and direct the engineer to `/plan-tasks` rather than inventing them. **Treat repository artifacts as authoritative; never depend on a previous conversation to know the task.**
2. Confirm the task exists and has been approved; review only linked acceptance criteria and relevant sections in original specifications. Check source version/hash if recorded. If a source changed, stop and ask to re-plan. Use targeted file search; do not dump the entire repo, unrelated history, generated files or long logs into context.
3. Inspect repository and `git status --short`, current branch and test commands. Do not overwrite/mix preexisting uncommitted work or silently switch/create a branch.
4. Read task statuses in `plan.md`:
   - If any other task is still `READY FOR HUMAN REVIEW`, ask the engineer in one question which of those tasks they have reviewed and accepted. Immediately set each confirmed task to `DONE`: the answer is the approval for those status lines, and this is the only edit allowed in Phase A. Leave unconfirmed tasks unchanged.
   - Every dependency of the selected task must be `DONE`. Otherwise **STOP** and name the unfinished dependencies.
   - The selected task must be `NOT STARTED` or `BLOCKED — <reason>`; resume a blocked task only when its reason is resolved. If it is `READY FOR HUMAN REVIEW` or `DONE`, **STOP** and ask whether to rework it.
5. If plan/decisions/code conflict, a newly discovered requirement changes the approach or the task is too large for one reviewable commit, **STOP**. Explain the smallest required plan/decision amendment and request approval; never silently redesign or expand scope.
6. Present a **short execution proposal**: exact scope, likely files, relevant tests/specialized checks, expected doc updates and risks.
7. If step 4 marked earlier tasks `DONE` and Git is initialized, check `git diff` for `plan.md`. When those status lines are its only uncommitted changes, recommend committing `plan.md` on its own before approving the proposal (for example, `docs: mark S02 as done`), so the next task's diff holds only its own work. This is optional; do not stage or commit it yourself. If `plan.md` has other uncommitted changes, do not recommend the separate commit; say that the status changes will be part of this task's diff instead.
8. **STOP for explicit approval before editing files.**

### Phase B — Implement (only after approval)

1. Implement only the selected task, following repository conventions and approved decisions.
2. Add/update meaningful unit and integration tests for the behavior. Test failure cases and edge conditions relevant to this task. Avoid unrelated refactors and unnecessary dependencies.
3. Keep docs aligned with code, including API/configuration/usage examples and accepted changes. If new design decisions become necessary, set the task to `BLOCKED — <reason>` and stop for approval before deviating from the plan.
4. Preserve all unrelated user changes.

### Phase C — Mandatory verification

1. **After implementation is complete**, run the repository’s **full unit test suite** and **full integration test suite**. Discover the actual commands from project documentation, build configuration, scripts or CI; this instruction applies to **any programming language or testing framework**. Execute the project-specific commands directly; CodeHelm provides no wrapper script. You may run targeted tests earlier for feedback, but they never replace these post-implementation full-suite runs. If either suite or its command is missing or cannot run, report the blocker and do not mark the task ready for commit.
2. Run build/static checks and task-specific checks listed in the plan (e.g., container build + smoke test, workflow lint, metrics endpoint test, DB migration test). Capture concise success summaries and full relevant failure diagnostics; avoid unnecessary large tool output.
3. Inspect `git diff --check`, `git diff` and `git status --short`. Validate the scope and compare implemented behavior and tests against linked requirements and approved `decisions.md`.
4. If any mandatory verification cannot run or fails, report the exact blocker and set the task to `BLOCKED — <reason>`; do **not** label it ready for commit. Do not invent results. Hosted CI success must be checked after the approved push; a local YAML lint is not equivalent.
5. Only when every mandatory check passed, set the task to `READY FOR HUMAN REVIEW` (not committed). In both cases, add executed test evidence to `verification.md` as *provisional*, awaiting independent audit. Do not add rows to its audit log; only `/audit` does. Never set `DONE` for the task just implemented; only the engineer accepts it, through `--done` or at the start of a later run. Do not mark requirements VERIFIED merely because code exists.

### Phase D — Hand off

Report concisely:
- What changed, why and how it works (explain important code paths)
- Files changed, key trade-offs and any documentation changes
- Exact unit/integration/specialized test commands and observed outcomes
- Known limitations and requirements needing audit
- Suggested single Conventional Commit message
- Suggested review commands: `git diff --check`, `git diff`, `git status --short`
- **Fresh-session handoff:** plan name + task ID + approved decisions referenced, task status (`READY FOR HUMAN REVIEW` or `BLOCKED — <reason>`), exact checks and outcomes, pending human action and the next command. If earlier tasks were marked `DONE` in this run and those status changes were not committed separately, say that this diff also contains them. For a ready task, recommend this order: review, optionally `/audit <plan-name> <task-id>`, then `/implement <plan-name> <task-id> --done`, then commit. Save lasting decisions and verification evidence in Markdown, not only chat.

**STOP.** Wait for the engineer to review, stage and commit manually. Never stage, commit, push or start the next task without fresh explicit approval.
