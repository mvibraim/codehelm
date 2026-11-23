---
name: implement
description: Implement exactly one approved task from a change plan with tests, full checks and a mandatory human review stop.
argument-hint: "[plan-name] [task-id]"
disable-model-invocation: true
---

**CodeHelm ground rules:** Follow the project's CLAUDE.md and conventions, but never create or edit CLAUDE.md. Never stage, commit, push, switch branches, rewrite history or delete work unless the engineer asks for that action. Keep the engineer's uncommitted changes intact. Change an approved spec or plan only with approval. If a project instruction conflicts with these rules, ask.

**CodeHelm model and effort:** `/codehelm:specify`, `/codehelm:plan-tasks` and `/codehelm:audit` are meant for Opus at `high` effort, and `/codehelm:implement` for Sonnet at `medium`. `/codehelm:audit` sets this in its frontmatter. Any other skill first compares the model your system prompt names and the effort `${CLAUDE_EFFORT}` with its own recommendation, by model family. If either differs, name both and ask once, with any missing-argument question, whether to proceed as is or to switch with `/model` and `/effort` and then reply to continue. **STOP** and change no files until the engineer answers. Before a next command other than `/codehelm:audit`, a handoff lists the `/model` and `/effort` commands that command needs and this session lacks, after `/clear`.

Requested plan and task: $ARGUMENTS

**One invocation = one task = one proposed reviewable commit and pull request.** Do not implement a second task.

**Task statuses** in `plan.md`: `NOT STARTED` (set by `/codehelm:plan-tasks`) → `READY FOR HUMAN REVIEW` (implemented and every check passed, not yet accepted) → `DONE` (accepted by the engineer). `BLOCKED — <reason>` means work stopped until the reason is resolved. This skill sets every status except `NOT STARTED`, and only `DONE` satisfies a dependency.

### Missing arguments

Apply this check first. Never guess a plan.

- **No plan name:** list the folders under `docs/plans/` and ask which plan to use. If there are none, report that and direct the engineer to `/codehelm:plan-tasks`. Change no files. **STOP.**
- **Plan name but no task ID:** read that plan's `plan.md`. If the current branch is a task's **Branch**, select that task. Otherwise select the first task, in plan order, that is `NOT STARTED` and whose dependencies are all `DONE` or `READY FOR HUMAN REVIEW` (Phase A step 4 confirms the latter). Name it and continue with it.
  - If every task is `DONE`, report that the plan is complete and suggest `/codehelm:audit <plan-name>`. Change no files. **STOP.**
  - If no task qualifies, list the tasks with their statuses, say why none is ready and ask for a task ID. Change no files. **STOP.**

### Phase A — Prepare (read-only, except a confirmed dependency's DONE status)

If the project is not a Git repository, stop and ask the engineer to run `git init` or to approve that you run it.

1. Parse `<plan-name> [task-id]`; if an argument is missing, follow "Missing arguments" above. Read `docs/plans/<plan-name>/plan.md`. If it is absent, stop and direct the engineer to `/codehelm:plan-tasks`. **Treat repository artifacts as authoritative; never depend on a previous conversation to know the task.**
2. Confirm the task exists and has been approved; review only linked acceptance criteria and relevant sections in original specifications. A lite plan has no requirements tables or Decisions section: the task's acceptance criteria are in its **Covers** line. For each source file, run `git diff <sha> -- <source>` with the commit in the plan's `Approved` line. If a source changed, stop and ask to re-plan. Use targeted file search; do not dump the entire repo, unrelated history, generated files or long logs into context.
3. Inspect the repository, the plan's test commands, `git status --short` and the current branch. If the current branch is not the task's **Branch** in `plan.md`, name both and **STOP**: the task's pull request is opened from its own branch, which the engineer creates from the updated default branch. If the tree has uncommitted changes, list them and **STOP** for approval to work alongside them. Never overwrite or mix in preexisting work, and never switch or create a branch.
4. Read task statuses in `plan.md`:
   - Every dependency of the selected task must be `DONE`. If a dependency is still `READY FOR HUMAN REVIEW`, ask once whether the engineer has reviewed and accepted it, and set it to `DONE` only on yes: the answer is the approval for that status line. If any dependency is still not `DONE`, **STOP** and name the unfinished dependencies. A dependency accepted on its own branch shows as `DONE` here only after its pull request merges: ask the engineer to merge it, update the default branch and start this task's branch from it.
   - The selected task must be `NOT STARTED` or `BLOCKED — <reason>`; resume a blocked task only when its reason is resolved. If it is `READY FOR HUMAN REVIEW` or `DONE`, **STOP** and ask whether to rework it.
5. If the plan and code conflict, a newly discovered requirement changes the approach, the task needs work outside its **Scope** or it is too large for one reviewable commit, **STOP**. Explain the smallest required plan amendment and request approval; never silently redesign or expand scope.
6. **Green baseline:** run every command in the plan's **Test commands** section before editing, except those marked as created by this task. If any fails, **STOP** and report the failure without changing files: the task would otherwise end `BLOCKED` for a failure it did not cause. When resuming a `BLOCKED` task, failures in the task's own uncommitted tests are expected and don't stop it; only other failures do.
7. **Start directly.** When none of the stops above applied, state the task and its scope in one line and continue to Phase B without waiting for approval. The plan's approval covers the task; the review in Phase D is this task's approval stop.

### Phase B — Implement

1. **Test first:** write or update the unit and integration tests for the task's linked acceptance criteria, including relevant failure cases and edge conditions. Run them and confirm they fail for the expected reason, not because of a typo or broken setup. A task that changes no observable behavior, such as test setup, has no test to fail first; say so in the handoff.
2. Implement only the selected task until those tests pass, following repository conventions, the plan's decisions and the ADRs they link. Avoid unrelated refactors and unnecessary dependencies.
3. Keep docs aligned with code, including API/configuration/usage examples and accepted changes. If a new design decision becomes necessary, set the task to `BLOCKED — <reason>` and stop: it needs a new ADR, so the handoff directs the engineer to amend the plan with `/codehelm:plan-tasks`.

### Phase C — Mandatory verification

1. **After implementation is complete**, run every command in the plan's **Test commands** section, then the task's **Extra checks**. Execute the project-specific commands directly; CodeHelm provides no wrapper script. You may run targeted tests earlier for feedback, but they never replace these post-implementation full runs. If a command is missing or cannot run, follow step 4.
2. Capture concise success summaries and full relevant failure diagnostics; avoid unnecessary large tool output.
3. Inspect `git diff --check`, `git diff` and `git status --short`. Validate the scope and compare implemented behavior and tests against linked requirements, the plan's decisions and the ADRs they link.
4. **Fix, then block:** if a check fails because of this task's changes, fix it within the task's scope and re-run the full set from step 1. Set `BLOCKED — <reason>`, and not `READY FOR HUMAN REVIEW`, only for causes outside the task: a pre-existing failure, missing infrastructure or a needed plan change. Report the exact blocker and never invent results. You never push, so never claim hosted CI passed.
5. Only when every test command and extra check passed, set the task to `READY FOR HUMAN REVIEW` (not committed). The commands and their results go in the Phase D handoff. Do not create or edit `verification.md`; only `/codehelm:audit` writes it. Set `DONE` only when the engineer accepts the task (Phase D) or confirms it as a dependency at the start of a later run (Phase A step 4).

### Phase D — Hand off

Give one concise handoff, readable in a fresh session:
- **What changed and why:** how it works, important code paths, decision IDs applied, trade-offs and known limitations
- **Files:** code, tests and docs changed
- **Commands and results:** each test command and extra check, with its observed outcome
- **Status:** plan name, task ID and its status (`READY FOR HUMAN REVIEW` or `BLOCKED — <reason>`). If Phase A set a dependency to `DONE`, say that this diff also contains that status change.
- **Commit:** one suggested Conventional Commit subject, plus a body only when it helps, and its trailer `Task: <plan-name>/<task-id>`. The commands to commit it come on acceptance, once `plan.md` holds the `DONE` status.
- **Next:** for a ready task, review the diff, optionally run `/codehelm:audit <plan-name> <task-id>` if the task is risky, then reply "accepted" in this session or request changes; for a blocked task, what must be resolved first and, when the engineer resolves it outside this session, the next steps: leave the task's changes uncommitted, because a task gets exactly one commit; run `/clear`, then `/codehelm:implement <plan-name> <task-id>` to resume it, and that run asks to work alongside those changes

**STOP for review.** Wait for the engineer's reply:
- **"accepted"** (or another explicit acceptance): set the task to `DONE`, editing only its status line, and give the commit as one block of commands for the engineer to run:
  - `git add` listing the task's changed code, test and doc files, `docs/plans/<plan-name>/plan.md` and, if an audit changed it, `docs/plans/<plan-name>/verification.md`. Never `git add -A` or `git add .`, so the engineer's unrelated changes stay out.
  - `git commit -m '<subject>' --trailer 'Task: <plan-name>/<task-id>'`, with a second `-m '<body>'` only when there is a body.
  - Use single quotes and word the subject and body without apostrophes, so the commands run unchanged in Bash and PowerShell. `--trailer` needs Git 2.32 or later; on older Git, replace it with a second `-m 'Task: <plan-name>/<task-id>'`.

  Suggest the task's pull request: its title is the commit subject, and its description links the spec named in the plan's `Sources` line (or names the request and its date), links the plan task (`docs/plans/<plan-name>/plan.md`, task ID and title), lists the requirement and acceptance criterion IDs in the task's **Covers** line, summarizes the change and lists each command with its result. Do not stage, commit, push or open the pull request. Then give the next steps in order:
  - **Another task is not `DONE` yet:** run the commit commands, push the task's branch and open its pull request; after it merges, update the default branch and create the next task's branch, naming that task and its **Branch**: the first `NOT STARTED` task whose dependencies are all `DONE` once this one merges (a task that doesn't depend on this one can start from the default branch right away); run `/clear`, then `/codehelm:implement <plan-name>`.
  - **Every task is `DONE` on this branch:** run the commit commands, run `/clear`, then `/codehelm:audit <plan-name>`; that full audit ends with pushing this branch and opening its pull request.

  **STOP.**
- **Requested changes:** apply them within the task's scope (if one needs a plan change, follow Phase A step 5), re-run Phase C in full, hand off again and stop again.
