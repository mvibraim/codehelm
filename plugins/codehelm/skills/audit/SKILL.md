---
name: audit
description: Independently audit a planned change against source specifications, accepted decisions, code, tests and actual verification evidence.
argument-hint: "[plan-name] [optional task-id]"
disable-model-invocation: true
context: fork
background: false
---

**CodeHelm ground rules:** Follow the project's CLAUDE.md and conventions, but never create or edit CLAUDE.md. Never stage, commit, push, switch branches, rewrite history or delete work unless the engineer asks for that action. Keep the engineer's uncommitted changes intact. Change an approved spec or plan only with approval. If a project instruction conflicts with these rules, ask.

Plan and optional task: $ARGUMENTS

**Purpose:** An evidence-based consistency audit; do not fix production code or change Git state. Report gaps that affect correctness, and skip style.

**Two modes:**
- **Full change audit:** `/codehelm:audit <plan-name>`: check all in-scope requirements and tasks, integration boundaries, architectural consistency and delivery readiness. Recommended at milestones and before delivery.
- **Task audit (optional):** `/codehelm:audit <plan-name> <task-id>`: inspect only the task, its linked requirements, changed files, tests and decisions. Meant for risky tasks, before the engineer accepts them.

This skill runs in a forked subagent without the conversation history, which keeps the audit independent of the implementation discussion. It cannot ask questions mid-run: wherever it must ask, it ends the run with the question and changes no files.

**No plan name:** if `$ARGUMENTS` is empty, list the folders under `docs/plans/` and end the run by asking the engineer to rerun `/codehelm:audit <plan-name> [task-id]` with one of them. Never pick one yourself, even when only one plan exists. If there are none, report that and direct the engineer to `/codehelm:plan-tasks`. **STOP** and change no files.

### Phase A — Gather evidence (read-only)

If the project is not a Git repository, stop and ask the engineer to run `git init` and then rerun the audit.

1. Read `docs/plans/<plan-name>/plan.md`, and its `verification.md` if an earlier audit created it. If the plan is missing, report the gap rather than fabricating a plan. For a **task audit**, read only linked specification sections, relevant source code/diff, tests and relevant API/config docs. Find the task's changes with `git status --short`, `git diff` and `git log`. For a **full audit**, inspect every in-scope requirement and cross-task interaction. Avoid indiscriminate whole-repository reads.
2. For each source file, run `git diff <sha> -- <source>` with the commit in the plan's `Approved` line. Flag changes rather than silently treating the old plan as current.
3. Read actual test results/reports or execute the plan's **Test commands** and the tasks' **Extra checks** if needed and permitted. Never treat a test name or claim that tests passed as proof. Record exact commands, output summary and date/revision when available.
4. Look for requirements without tasks, tasks without source justification, undocumented implementation, deviations from accepted decisions, missing edge-case coverage, test assertions that do not actually validate the business rule, and test commands that miss a suite the changes need (for example, a plan approved without integration tests whose changes now cross an integration boundary).
5. Read each in-scope task's status in `plan.md` (`NOT STARTED`, `BLOCKED — <reason>`, `READY FOR HUMAN REVIEW` when implemented with every check passed, or `DONE` once the engineer accepted it) and compare it with the evidence. Report a status the evidence contradicts as a finding, such as `DONE` or `READY FOR HUMAN REVIEW` with missing changes or failing checks, or `NOT STARTED` with existing code. Never change a task status yourself.
6. For infrastructure/CI/observability changes, inspect relevant specialized checks (container smoke, workflow run, emitted metric tests, migrations). If live checks cannot be executed, mark appropriately.

### Phase B — Verify each criterion

For every applicable requirement and acceptance criterion, record:
- Stable requirement/criterion ID, as in `plan.md`
- Planned task(s)
- Actual source file/symbol evidence
- Test file/case, with the executed command and its result
- Status: `VERIFIED`, `FAILED`, `NOT VERIFIED` or `NOT APPLICABLE`

Check documentation and decision alignment too, and record gaps as findings.

Use `VERIFIED` only for supported behavior with adequate executed evidence. `FAILED` means demonstrated contradiction/failure. `NOT VERIFIED` means missing, blocked or insufficient evidence. `NOT APPLICABLE` requires explanation.

### Phase C — Report and document

1. Prioritize findings by severity; cite precise file locations and requirement IDs. Separate proven failures from unverified concerns and propose proportionate fixes.
2. Identify missing/weak tests, unapproved design decisions, stale documents and implementation drift.
3. Update **only** `docs/plans/<plan-name>/verification.md` to record grounded findings, evidence and unresolved issues. On the first audit of a plan, create it from the bundled template `${CLAUDE_SKILL_DIR}/assets/verification.md`. Task statuses in `plan.md` stay unchanged; record status mismatches as findings instead. Preserve previous meaningful history; don't erase open findings without justification.
   - **Evidence:** one row per in-scope requirement and acceptance criterion, as described in Phase B, updated in place.
   - **Findings:** one row per finding, with its severity, the requirement ID or task, the evidence, the next action and its status.
4. Append one row to the `## Audit log` table in `verification.md`: date, audited revision (git SHA, or `uncommitted working tree` when uncommitted changes are in scope), scope (the task ID or `full`), a one-line summary and the open findings counted by severity after this run, for example `3 (1 high, 2 medium)` or `0`. Never edit or remove earlier rows.
5. Propose next small, reviewable correction tasks where needed; do not implement them and do not silently change the approved plan.
6. Summarize whether the **audited scope** is ready for human review / final delivery and what remains blocked. A task-level pass must never be presented as a full-change pass. Provide a concise fresh-session handoff that points to durable docs and outstanding findings, and suggest committing `verification.md` before the next task, with the task's changes after a task audit.

Never claim full coverage from passing tests alone.
