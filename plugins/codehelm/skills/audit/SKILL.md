---
name: audit
description: Independently audit a planned change against source specifications, accepted decisions, code, tests and actual verification evidence.
argument-hint: "[plan-name] [optional task-id]"
disable-model-invocation: true
context: fork
background: false
model: opus
effort: high
---

**CodeHelm ground rules:** Never create or edit CLAUDE.md. Never stage, commit, push, switch branches, rewrite history or delete work unless the engineer asks. Keep the engineer's uncommitted changes. Change an approved spec or plan only with approval. If a project instruction conflicts with these rules, ask.

**CodeHelm model and effort:** `/codehelm:specify`, `/codehelm:plan-tasks` and `/codehelm:audit`: Opus at `high`. `/codehelm:implement`: Sonnet at `medium`. If this session differs (the model family your system prompt names, effort `${CLAUDE_EFFORT}`), say so in one line and continue. A handoff that ends a skill's work ends with a **Next** block, one step per line: the commit commands (single-quoted and without apostrophes, so they run in Bash and PowerShell), any pull request step, `/clear`, the `/model` and `/effort` the next command needs and this session lacks (`/codehelm:audit` sets its own), then the next command.

Plan and optional task: $ARGUMENTS

**Purpose:** An evidence-based consistency audit; do not fix production code or change Git state. Report gaps that affect correctness, and skip style.

- **Full audit** (`/codehelm:audit <plan-name>`): every in-scope requirement and task, cross-task interactions, architectural consistency and delivery readiness.
- **Task audit** (`/codehelm:audit <plan-name> <task-id>`): only the task, its linked requirements and spec sections, its changes, tests and decisions.

You can't ask mid-run: wherever you must ask, end the run with the question and change no files.

**No plan name:** if `$ARGUMENTS` is empty and `docs/plans/` holds exactly one plan folder, run a full audit of that plan and name it in the report. If it holds several, list them and end the run by asking the engineer to rerun `/codehelm:audit <plan-name> [task-id]` with one of them. If there are none, report that and direct the engineer to `/codehelm:plan-tasks`. In both cases, change no files.

### Phase A — Gather evidence (read-only)

If the project is not a Git repository, stop and ask the engineer to run `git init` and then rerun the audit.

1. Read `docs/plans/<plan-name>/plan.md`, the ADRs it links, and its `verification.md` if an earlier audit created it. If the plan is missing, report the gap rather than fabricating a plan. The requirements to audit are the spec's on the plan's `Sources` line, except those the plan leaves out of scope, plus the plan's Requirements table; each task's **Covers** line links them. Find a task's commits with `git log --grep "Task: <plan-name>/<task-id>"` (plain `git log` for commits without the trailer), and its uncommitted work with `git status --short` and `git diff`.
2. For each source file, run `git diff <sha> -- <source>` with the commit in the plan's `Approved` line. Flag changes rather than silently treating the old plan as current.
3. Read actual test results/reports or execute the plan's **Test commands** and the tasks' **Extra checks** if needed and permitted. Never treat a test name or claim that tests passed as proof. Record exact commands, output summary and date/revision when available.
4. Look for requirements without tasks, tasks without source justification, undocumented implementation, stale documentation, code that contradicts an accepted decision or ADR, a linked ADR that is missing or superseded, missing edge-case coverage, test assertions that do not actually validate the business rule, and test commands that miss a suite the changes need (for example, a plan approved without integration tests whose changes now cross an integration boundary).
5. Compare each in-scope task's status in `plan.md` with the evidence. Report a status the evidence contradicts as a finding, such as `DONE` or `READY FOR HUMAN REVIEW` with missing changes or failing checks, or `NOT STARTED` with existing code. Never change a task status yourself.

### Phase B — Verify each criterion

Give every applicable requirement and acceptance criterion an Evidence row, by its ID in the spec or `plan.md`, with source file/symbol and test case evidence. Use `VERIFIED` only for supported behavior with adequate executed evidence. `FAILED` means demonstrated contradiction/failure. `NOT VERIFIED` means missing, blocked or insufficient evidence. `NOT APPLICABLE` requires explanation.

### Phase C — Report and document

1. Prioritize findings by severity; cite precise file locations and requirement IDs. Separate proven failures from unverified concerns and propose proportionate fixes.
2. Update **only** `docs/plans/<plan-name>/verification.md`, created on the first audit of a plan from the bundled template `${CLAUDE_SKILL_DIR}/assets/verification.md`: one Findings row per finding and the Evidence rows updated in place. Don't erase open findings without justification.
3. Append one Audit log row. Revision is the git SHA, or `uncommitted working tree` when uncommitted changes are in scope; Scope is the task ID or `full`; Open findings counts them by severity after this run, such as `3 (1 high, 2 medium)` or `0`. Never edit or remove earlier rows.
4. Propose next small, reviewable correction tasks where needed; do not implement them and do not silently change the approved plan.
5. Summarize whether the **audited scope** is ready for human review / final delivery and what remains blocked. A task-level pass must never be presented as a full-change pass. Provide a concise fresh-session handoff that points to durable docs and outstanding findings, then:
   - **Full audit:** **Next:** `git switch -c docs/audit-<plan-name>` only when on the default branch, `git add docs/plans/<plan-name>/verification.md`, `git commit -m 'docs: audit <plan-name>'`. When the change is ready for delivery, end with pushing the last task's branch and opening its pull request; otherwise, on `docs/audit-<plan-name>`, push and open its pull request; then `/clear` and the next command, such as `/codehelm:implement <plan-name>`.
   - **Task audit:** no Next block: return to the task's review stop without `/clear`; `verification.md` ships in the task's commit.
