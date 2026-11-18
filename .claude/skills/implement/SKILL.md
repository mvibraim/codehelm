---
name: implement
description: Implement exactly one approved task from a change plan with tests, document checks and a mandatory human review stop.
argument-hint: "[plan-name] [task-id]"
disable-model-invocation: true
---

**CodeHelm portability and control:** This skill is self-contained. Respect an existing project `CLAUDE.md` when present; never create, copy, replace or merge one as a prerequisite. Follow existing repository conventions and preserve the engineer's uncommitted work. Never stage, commit, push, create/switch branches, rewrite history or perform destructive changes without explicit approval for that action. Project specifications and approved decisions are durable sources of truth; do not silently change them. If project instructions conflict with a CodeHelm safeguard, stop and ask the engineer to resolve the conflict. If Git is not initialized, report that fact and never initialize it without approval.

Requested plan and task: $ARGUMENTS

**One invocation = one task = one proposed reviewable commit.** Do not implement a second task or automatically commit.

### Phase A — Prepare (read-only)

1. Parse `<plan-name> <task-id>`; if missing, ask for the exact IDs. Read the project's `CLAUDE.md` **only if it exists**; CodeHelm does not provide one. Read `docs/plans/<plan-name>/plan.md`, `decisions.md` and `verification.md`. If the plan artifacts are absent, stop and direct the engineer to `/plan-tasks` rather than inventing them. **Treat repository artifacts as authoritative; never depend on a previous conversation to know the task.**
2. Confirm the task exists and has been approved; review only linked acceptance criteria and relevant sections in original specifications. Check source version/hash if recorded. If a source changed, stop and ask to re-plan. Use targeted file search; do not dump the entire repo, unrelated history, generated files or long logs into context.
3. Inspect repository and `git status --short`, current branch, previous task completion and test commands. Do not overwrite/mix preexisting uncommitted work or silently switch/create a branch.
4. If plan/decisions/code conflict, a newly discovered requirement changes the approach or the task is too large for one reviewable commit, **STOP**. Explain the smallest required plan/decision amendment and request approval; never silently redesign or expand scope.
5. Present a **short execution proposal**: exact scope, likely files, relevant tests/specialized checks, expected doc updates and risks.
6. **STOP for explicit approval before editing files.**

### Phase B — Implement (only after approval)

1. Implement only the selected task, following repository conventions and approved decisions.
2. Add/update meaningful unit and integration tests for the behavior. Test failure cases and edge conditions relevant to this task. Avoid unrelated refactors and unnecessary dependencies.
3. Keep docs aligned with code, including API/configuration/usage examples and accepted changes. If new design decisions become necessary, stop for approval before deviating from the plan.
4. Preserve all unrelated user changes.

### Phase C — Mandatory verification

1. **After implementation is complete**, run the repository’s **full unit test suite** and **full integration test suite**. Discover the actual commands from project documentation, build configuration, scripts or CI; this instruction applies to **any programming language or testing framework**. Execute the project-specific commands directly; CodeHelm provides no wrapper script. You may run targeted tests earlier for feedback, but they never replace these post-implementation full-suite runs. If either suite or its command is missing or cannot run, report the blocker and do not mark the task ready for commit.
2. Run build/static checks and task-specific checks listed in the plan (e.g., container build + smoke test, workflow lint, metrics endpoint test, DB migration test). Capture concise success summaries and full relevant failure diagnostics; avoid unnecessary large tool output.
3. Inspect `git diff --check`, `git diff` and `git status --short`. Validate the scope and compare implemented behavior and tests against linked requirements and approved `decisions.md`.
4. If any mandatory verification cannot run or fails, report the exact blocker; do **not** label the step ready for commit. Do not invent results. Hosted CI success must be checked after the approved push; a local YAML lint is not equivalent.
5. Update task status as `READY FOR HUMAN REVIEW` (not committed) and add executed test evidence to `verification.md` as *provisional*, awaiting independent audit. Do not mark requirements VERIFIED merely because code exists.

### Phase D — Hand off

Report concisely:
- What changed, why and how it works (explain important code paths)
- Files changed, key trade-offs and any documentation changes
- Exact unit/integration/specialized test commands and observed outcomes
- Known limitations and requirements needing audit
- Suggested single Conventional Commit message
- Suggested review commands: `git diff --check`, `git diff`, `git status --short`
- **Fresh-session handoff:** plan name + task ID + approved decisions referenced, task state (`READY FOR HUMAN REVIEW` or blocked), exact checks and outcomes, pending human action and the next command. Save lasting decisions and verification evidence in Markdown, not only chat.

**STOP.** Wait for the engineer to review, stage and commit manually. Never stage, commit, push or start the next task without fresh explicit approval.
