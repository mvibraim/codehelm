---
name: implement
description: Implement exactly one approved task from a change plan with tests, full checks and a mandatory human review stop.
argument-hint: "[plan-name] [task-id]"
disable-model-invocation: true
---

**CodeHelm ground rules:** Follow the project's CLAUDE.md and conventions, but never create or edit CLAUDE.md. Never stage, commit, push, switch branches, rewrite history or delete work unless the engineer asks for that action. Keep the engineer's uncommitted changes intact. Change an approved spec or plan only with approval. If a project instruction conflicts with these rules, ask.

Requested plan and task: $ARGUMENTS

**One invocation = one task = one proposed reviewable commit.** Do not implement a second task or automatically commit.

**Task statuses** in `plan.md`: `NOT STARTED` (set by `/codehelm:plan-tasks`) → `READY FOR HUMAN REVIEW` (implemented and every check passed, not yet accepted) → `DONE` (accepted by the engineer). `BLOCKED — <reason>` means work stopped until the reason is resolved. This skill sets every status except `NOT STARTED`, and only `DONE` satisfies a dependency.

### Missing arguments

Apply this check first. Never guess a plan.

- **No plan name:** list the folders under `docs/plans/` and ask which plan to use. If there are none, report that and direct the engineer to `/codehelm:plan-tasks`. Change no files. **STOP.**
- **Plan name but no task ID:** read that plan's `plan.md` and select the first task, in plan order, that is `NOT STARTED` and whose dependencies are all `DONE` or `READY FOR HUMAN REVIEW` (Phase A step 4 confirms the latter). Name it and continue with it. If every task is `DONE`, report that the plan is complete and suggest `/codehelm:audit <plan-name>`. If no task qualifies, list the tasks with their statuses, say why none is ready and ask for a task ID. **STOP.**

### Phase A — Prepare (read-only, except a confirmed dependency's DONE status)

If the project is not a Git repository, stop and ask the engineer to run `git init` or to approve that you run it.

1. Parse `<plan-name> [task-id]`; if an argument is missing, follow "Missing arguments" above. Read `docs/plans/<plan-name>/plan.md`. If it is absent, stop and direct the engineer to `/codehelm:plan-tasks` rather than inventing them. **Treat repository artifacts as authoritative; never depend on a previous conversation to know the task.**
2. Confirm the task exists and has been approved; review only linked acceptance criteria and relevant sections in original specifications. For each source file, run `git diff <sha> -- <source>` with the commit in the plan's `Approved` line. If a source changed, stop and ask to re-plan. Use targeted file search; do not dump the entire repo, unrelated history, generated files or long logs into context.
3. Inspect the repository, the plan's test commands, `git status --short` and the current branch. If the tree has uncommitted changes, list them and **STOP** for approval to work alongside them. Never overwrite or mix in preexisting work, and never switch or create a branch.
4. Read task statuses in `plan.md`:
   - Every dependency of the selected task must be `DONE`. If a dependency is still `READY FOR HUMAN REVIEW`, ask once whether the engineer has reviewed and accepted it, and set it to `DONE` only on yes: the answer is the approval for that status line. This is the only edit allowed in Phase A. If any dependency is still not `DONE`, **STOP** and name the unfinished dependencies.
   - The selected task must be `NOT STARTED` or `BLOCKED — <reason>`; resume a blocked task only when its reason is resolved. If it is `READY FOR HUMAN REVIEW` or `DONE`, **STOP** and ask whether to rework it.
5. If the plan and code conflict, a newly discovered requirement changes the approach, the task needs work outside its **Scope** or it is too large for one reviewable commit, **STOP**. Explain the smallest required plan amendment and request approval; never silently redesign or expand scope.
6. **Green baseline:** run every command in the plan's **Test commands** section before editing, except those marked as created by this task. If any fails, **STOP** and report the failure without changing files: the task would otherwise end `BLOCKED` for a failure it did not cause.
7. **Start directly.** When none of the stops above applied, state the task and its scope in one line and continue to Phase B without waiting for approval. The plan's approval covers the task; the review in Phase D is this task's approval stop.

### Phase B — Implement

1. **Test first:** write or update the unit and integration tests for the task's linked acceptance criteria, including relevant failure cases and edge conditions. Run them and confirm they fail for the expected reason, not because of a typo or broken setup. A task that changes no observable behavior, such as test setup, has no test to fail first; say so in the handoff.
2. Implement only the selected task until those tests pass, following repository conventions, the plan's decisions and the ADRs they link. Avoid unrelated refactors and unnecessary dependencies.
3. Keep docs aligned with code, including API/configuration/usage examples and accepted changes. If a new design decision becomes necessary, set the task to `BLOCKED — <reason>` and stop: it needs a new ADR, so the handoff directs the engineer to amend the plan with `/codehelm:plan-tasks`.
4. Preserve all unrelated user changes.

### Phase C — Mandatory verification

1. **After implementation is complete**, run every command in the plan's **Test commands** section, then the task's **Extra checks**. Execute the project-specific commands directly; CodeHelm provides no wrapper script. You may run targeted tests earlier for feedback, but they never replace these post-implementation full runs. If a command is missing or cannot run, report the blocker and do not set `READY FOR HUMAN REVIEW` (see step 4).
2. Capture concise success summaries and full relevant failure diagnostics; avoid unnecessary large tool output.
3. Inspect `git diff --check`, `git diff` and `git status --short`. Validate the scope and compare implemented behavior and tests against linked requirements, the plan's decisions and the ADRs they link.
4. **Fix, then block:** if a check fails because of this task's changes, fix it within the task's scope and re-run the full set from step 1. Set `BLOCKED — <reason>`, and not `READY FOR HUMAN REVIEW`, only for causes outside the task: a pre-existing failure, missing infrastructure or a needed plan change. Report the exact blocker and never invent results. You never push, so never claim hosted CI passed.
5. Only when every test command and extra check passed, set the task to `READY FOR HUMAN REVIEW` (not committed). The status records the result: `READY FOR HUMAN REVIEW` means every check passed, and `BLOCKED — <reason>` says what failed. The commands and their results go in the Phase D handoff. Do not create or edit `verification.md`; only `/codehelm:audit` writes it. Set `DONE` only when the engineer accepts the task (Phase D) or confirms it as a dependency at the start of a later run (Phase A step 4).

### Phase D — Hand off

Give one concise handoff, readable in a fresh session:
- **What changed and why:** how it works, important code paths, decision IDs applied, trade-offs and known limitations
- **Files:** code, tests and docs changed
- **Commands and results:** each test command and extra check, with its observed outcome
- **Status:** plan name, task ID and its status (`READY FOR HUMAN REVIEW` or `BLOCKED — <reason>`). If Phase A set a dependency to `DONE`, say that this diff also contains that status change.
- **Commit message:** one suggested Conventional Commit message that ends, after a blank line, with the trailer `Task: <plan-name>/<task-id>`
- **Next:** for a ready task, review the diff, optionally run `/codehelm:audit <plan-name> <task-id>` if the task is risky, then reply "accepted" in this session or request changes; for a blocked task, what must be resolved first and, when the engineer resolves it outside this session, the next steps: run `/clear`, then `/codehelm:implement <plan-name> <task-id>` to resume it

**STOP for review.** Wait for the engineer's reply:
- **"accepted"** (or another explicit acceptance): set the task to `DONE`, editing only its status line, and repeat the suggested commit message with its `Task:` trailer, noting that `plan.md`, and `verification.md` if an audit changed it, belong in the commit. Do not stage or commit. Then give the next steps in order: commit, run `/clear`, then `/codehelm:implement <plan-name>` for the next ready task, or `/codehelm:audit <plan-name>` when every task is `DONE`. **STOP.**
- **Requested changes:** apply them within the task's scope (if one needs a plan change, follow Phase A step 5), re-run Phase C in full, hand off again and stop again.

If the session ends without a reply, the task stays `READY FOR HUMAN REVIEW`.
