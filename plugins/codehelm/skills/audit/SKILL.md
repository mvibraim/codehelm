---
name: audit
description: Independently audit a planned change against source specifications, accepted decisions, code, tests and actual verification evidence.
argument-hint: "[plan-name] [optional task-id]"
disable-model-invocation: true
---

**CodeHelm ground rules:** Follow the project's CLAUDE.md and conventions, but never create or edit CLAUDE.md. Never stage, commit, push, switch branches, rewrite history or delete work unless the engineer asks for that action. Keep the engineer's uncommitted changes intact. Change an approved spec or plan only with approval. If a project instruction conflicts with these rules, ask.

Plan and optional task: $ARGUMENTS

**Purpose:** An evidence-based consistency audit; do not fix production code or change Git state.

**Two modes:**
- **Task audit:** `/codehelm:audit <plan-name> <task-id>`: inspect only the task, its linked requirements, changed files, tests and decisions. Prefer this before each human-reviewed commit.
- **Full change audit:** `/codehelm:audit <plan-name>`: check all in-scope requirements and tasks, integration boundaries, architectural consistency and delivery readiness. Run at milestones and before delivery.

**No plan name:** if `$ARGUMENTS` is empty, list the folders under `docs/plans/` and ask which plan to audit, and optionally which task. Never pick one yourself, even when only one plan exists. If there are none, report that and direct the engineer to `/codehelm:plan-tasks`. **STOP** and change no files until the engineer answers.

For independence, prefer a fresh Claude session. Do not rely on the implementation conversation for evidence.

### Phase A — Gather evidence (read-only)

If the project is not a Git repository, stop and ask the engineer to run `git init` or to approve that you run it.

1. Read `docs/plans/<plan-name>/{plan,decisions,verification}.md`. If the plan artifacts are missing, report the gap rather than fabricating a plan. For a **task audit**, read only linked specification sections, relevant source code/diff, tests and relevant API/config docs. Find the task's changes with `git status --short`, `git diff` and `git log`. For a **full audit**, inspect every in-scope requirement and cross-task interaction. Avoid indiscriminate whole-repository reads.
2. Compare specification versions/hashes with those recorded in the plan. Flag changes rather than silently treating the old plan as current.
3. Read actual test results/reports or execute documented **unit + integration** suites if needed and permitted. Never treat a test name or claim that tests passed as proof. Record exact commands, output summary and date/revision when available.
4. Look for requirements without tasks, tasks without source justification, undocumented implementation, deviations from accepted decisions, missing edge-case coverage and test assertions that do not actually validate the business rule.
5. Read each in-scope task's status (task status lifecycle in `plan.md`) and compare it with the evidence. Report mismatches as findings, for example: `DONE` while the task's changes are missing or its checks fail; `READY FOR HUMAN REVIEW` with failing, missing or unrecorded checks; `NOT STARTED` while the task's code exists; `BLOCKED` without a reason; a task with code whose dependencies are not `DONE`; a `BLOCKED` reason caused only by a suite approved as `NOT APPLICABLE`. Never change a task status yourself.
6. For each unit or integration suite marked `NOT APPLICABLE` in an in-scope task, check that it has a reason and a recorded approval, and that the reason still holds for the task's actual changes (for example, a `docs` task that now changes code). Report an unjustified or unapproved `NOT APPLICABLE` as a finding; a suite skipped without approval is a missing mandatory check.
7. For infrastructure/CI/observability changes, inspect relevant specialized checks (container smoke, workflow run, emitted metric tests, migrations). If live checks cannot be executed, mark appropriately.

### Phase B — Verify each criterion

For every applicable requirement and acceptance criterion, record:
- Stable requirement/criterion ID and source reference
- Planned task(s)
- Actual source file/symbol evidence
- Test file/case and executed test result
- Documentation/decision alignment
- Recorded by: `audit` when this audit executed or independently checked the evidence; otherwise keep the existing value
- Status: `VERIFIED`, `FAILED`, `NOT VERIFIED` or `NOT APPLICABLE`

Use `VERIFIED` only for supported behavior with adequate executed evidence. `FAILED` means demonstrated contradiction/failure. `NOT VERIFIED` means missing, blocked or insufficient evidence. `NOT APPLICABLE` requires explanation.

Treat rows recorded by `implement (provisional)`, or with `—`, as claims to re-check, not as proof. Switch a matrix row to `audit` only after running its checks yourself (or reading a CI run or report for the audited revision) and then set its status from that evidence. If you cannot re-check it, leave the row `implement (provisional)` and `NOT VERIFIED`. A row may be `VERIFIED` only when it is recorded by `audit`.

### Phase C — Report and document

1. Prioritize findings by severity; cite precise file locations and requirement IDs. Separate proven failures from unverified concerns and propose proportionate fixes.
2. Identify missing/weak tests, unapproved design decisions, stale documents and implementation drift.
3. Update **only** `docs/plans/<plan-name>/verification.md` to record grounded findings, evidence and unresolved issues. Task statuses in `plan.md` stay unchanged; record status mismatches as findings instead. Preserve previous meaningful history; don't erase open findings without justification.
   - **Requirements evidence matrix:** update in-scope rows in place as described in Phase B.
   - **Executed checks:** append one row per check this audit ran or read, with the scope as `Task` (the task ID or `full`) and `Recorded by` = `audit`. Never edit or remove earlier rows, including provisional ones; replace only the `No checks recorded yet` placeholder row. If an older file lacks the `Task` or `Recorded by` column, add it and put `—` in existing rows.
4. Append one row to the `## Audit log` table in `verification.md`: date, audited revision (git SHA, or `uncommitted working tree` when uncommitted changes are in scope), scope (the task ID or `full`), a one-line summary and the open findings counted by severity after this run. Never edit or remove earlier rows; replace only the `No audits yet` placeholder row. If the file still has the older header fields `Task ID`, `Audited revision` or `Audit date`, move their values into a first log row (summary `Earlier audit, migrated from header`), remove those header fields, then append this run's row. If the file has no audit log, add the section after the statuses line.
5. Propose next small, reviewable correction tasks where needed; do not implement them and do not silently change the approved plan.
6. Summarize whether the **audited scope** is ready for human review / final delivery and what remains blocked. A task-level pass must never be presented as a full-change pass. Provide a concise fresh-session handoff that points to durable docs and outstanding findings.

Never claim full coverage from passing tests alone.
