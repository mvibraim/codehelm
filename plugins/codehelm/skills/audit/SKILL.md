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

**CodeHelm ground rules:** Never create or edit CLAUDE.md. Never stage, commit, push, switch branches, rewrite history or delete work unless the engineer asks, and keep their uncommitted changes. Change an approved spec or plan only with approval. If a project instruction conflicts with these rules, ask.

**CodeHelm model and effort:** `/codehelm:specify`, `/codehelm:plan-tasks` and `/codehelm:audit` run on Opus at `high`, `/codehelm:implement` on Sonnet at `medium`. If this session differs (your system prompt's model family, effort `${CLAUDE_EFFORT}`), say so in one line and continue. A handoff that ends a skill's work ends with a **Next** block, one step per line: commit commands (single-quoted, no apostrophes, so they run in Bash and PowerShell), any pull request step, `/clear`, the next command's `/model` and `/effort` (`/codehelm:audit` sets its own), then the next command.

Plan and optional task: $ARGUMENTS

Audit the change against evidence; don't fix code or change Git state. Report gaps that affect correctness, not style.

- **Full audit** (`/codehelm:audit <plan-name>`): every in-scope requirement and task, how tasks interact and delivery readiness.
- **Task audit** (`/codehelm:audit <plan-name> <task-id>`): the task, its requirements, changes, tests and decisions.

You can't ask mid-run: wherever you must ask, end the run with the question and change no files.

**No plan name:** with exactly one folder in `docs/plans/`, run a full audit of it and name it. With several, list them and end by asking for `/codehelm:audit <plan-name> [task-id]`. With none, point to `/codehelm:plan-tasks`.

### Phase A — Gather evidence (read-only)

If the project is not a Git repository, end the run asking the engineer to run `git init`.

1. Read `docs/plans/<plan-name>/plan.md`, its linked ADRs and any `verification.md`. If the plan is missing, report it. Audit the requirements of the spec on the `Sources` line, minus those out of scope, plus the plan's Requirements table. Find a task's commits with `git log --grep "Task: <plan-name>/<task-id>"` and uncommitted work with `git status --short` and `git diff`.
2. For each source, run `git diff <sha> -- <source>` with the commit on the `Approved` line, and flag any change.
3. Run the plan's **Test commands** and the tasks' **Extra checks**, or read their actual reports. A test name or a claim that tests passed is not proof. Record commands, output summary and revision.
4. Look for requirements without tasks, code without a requirement, stale docs, code that contradicts an accepted ADR, a linked ADR that is missing or superseded, missing edge cases, assertions that don't test the business rule, and test commands that miss a suite the change needs.
5. Report as a finding any task status the evidence contradicts, such as `DONE` with failing checks or `NOT STARTED` with code. Never change a task status.

### Phase B — Verify each criterion

Give each applicable requirement and acceptance criterion an Evidence row with its code and test evidence: `VERIFIED` only with adequate executed evidence, `FAILED` for a demonstrated failure, `NOT VERIFIED` for missing or insufficient evidence, `NOT APPLICABLE` with a reason.

### Phase C — Report

1. Rank findings by severity with file locations and requirement IDs, separate proven failures from unverified concerns, and propose a proportionate fix for each.
2. Write only `docs/plans/<plan-name>/verification.md`, created on the first audit from `${CLAUDE_SKILL_DIR}/assets/verification.md`: a Findings row per finding and the Evidence rows updated in place. Close an open finding only with a reason.
3. Append one Audit log row: Revision is the SHA or `uncommitted working tree`, Scope the task ID or `full`, Open findings the count by severity, such as `3 (1 high, 2 medium)` or `0`. Never edit earlier rows.
4. Propose small correction tasks where needed, without implementing them or changing the plan.
5. Say whether the audited scope is ready for review or delivery and what is blocked; never present a task pass as a full pass. Hand off with the docs and open findings, then:
   - **Full audit:** **Next:** `git switch -c docs/audit-<plan-name>` only when on the default branch, `git add docs/plans/<plan-name>/verification.md`, `git commit -m 'docs: audit <plan-name>'`. If the change is ready, push the last task's branch and open its pull request; otherwise push `docs/audit-<plan-name>` and open its pull request; then `/clear` and the next command, such as `/codehelm:implement <plan-name>`.
   - **Task audit:** no **Next** block: return to the task's review stop without `/clear`; `verification.md` ships in the task's commit.
