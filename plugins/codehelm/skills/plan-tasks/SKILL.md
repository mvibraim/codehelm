---
name: plan-tasks
description: Analyze a specification or engineering request, ask focused questions, document approved decisions and plan small reviewable tasks.
argument-hint: "[change request or paths to specifications]"
disable-model-invocation: true
---

**CodeHelm portability and control:** This skill is self-contained. Respect an existing project `CLAUDE.md` when present; never create, copy, replace or merge one as a prerequisite. Follow existing repository conventions and preserve the engineer's uncommitted work. Never stage, commit, push, create/switch branches, rewrite history or perform destructive changes without explicit approval for that action. Project specifications and approved decisions are durable sources of truth; do not silently change them. If project instructions conflict with a CodeHelm safeguard, stop and ask the engineer to resolve the conflict. If Git is not initialized, report that fact and never initialize it without approval.

Plan request and/or source documents: $ARGUMENTS

**Purpose:** Create a human-approved, requirements-traceable engineering plan. This skill is task-neutral: it applies to features, fixes, Docker, CI, observability, refactors and other changes.

**No input:** if `$ARGUMENTS` is empty, ask for a change request or the path to a specification, and list the Markdown files under `docs/specifications/`, if any, as options. Do not guess a request from the repository or the conversation. **STOP** and change no files until the engineer answers.

### Phase A — Discover and classify (read-only)

1. Read the project's `CLAUDE.md` **if present**, and respect its project-specific instructions. CodeHelm does not supply or require that file; when absent, work from this skill and the repository's existing conventions. Inspect relevant repository structure, existing design/docs, build/test commands and Git conventions. Check `git status --short` when Git is initialized; otherwise note the absence of Git and do not initialize it. Do not modify anything yet. **Prefer scoped searches and relevant file sections over loading the whole repository.**
2. Read all supplied specification documents completely enough to capture relevant rules. If a referenced file cannot be read, report that limitation instead of guessing. Do not alter original specifications. Check each specification's status field: if it is missing or not `Approved` (for example `Draft`), warn the engineer and carry the item into Phase B.
3. Choose a short `plan-name` and suggest a change type (`feat`, `fix`, `build`, `ci`, `chore`, `refactor`, `perf`, `test`, `docs`) and branch name. Do **not** create the branch.
4. Extract explicit functional requirements, business rules, nonfunctional constraints, error behavior and acceptance criteria. Preserve existing IDs and provenance labels unchanged; otherwise assign stable `FR-###`, `BR-###`, `NFR-###` and `AC-###` IDs. Cite source file + section/page where practical. Give every requirement and acceptance criterion exactly one basis label:
   - `EXPLICIT`: stated in a named source (a specification section, or the engineer's request with its date).
   - `ASSUMPTION`: not stated in a source, but approved by the engineer.
   - `PROPOSED`: inferred or suggested by you and not yet approved.
   If no specification exists, derive the list from the request: what the request states directly is `EXPLICIT` (source: the request), and everything you infer is `PROPOSED`. Each acceptance criterion names the requirement IDs it covers and states an observable outcome (Given/When/Then where practical).
5. Identify scope boundaries, contradictions, unknowns, dependencies, edge cases and source versions. Where practical record `git rev-parse HEAD` and source file SHA-256 fingerprints so later steps can detect specification drift.

### Phase B — Clarify (mandatory gate)

1. Ask a **small, prioritized group** of material questions; provide reasonable alternatives and your tentative recommendation.
2. Explicitly distinguish requirements from assumptions and engineering choices. List every `PROPOSED` requirement or acceptance criterion, including any still marked `PROPOSED` in a supplied specification, and ask the engineer to approve or reject each. An approved item becomes `ASSUMPTION`; a rejected item moves out of scope. Do not edit the specification; when the plan's label differs from the specification's, say so in the plan's source reference.
3. If a supplied specification is not `Approved` (Phase A step 2), ask the engineer whether to plan from it anyway or to finish it with `/codehelm:specify` first. Do not change its status yourself. When the engineer chooses to proceed, record its actual status next to its version in `plan.md`'s sources table.
4. **STOP and wait** for answers. Repeat only when essential ambiguities remain. Do not produce authoritative plan files yet.

### Phase C — Propose design and execution plan (read-only)

1. Recommend the simplest design consistent with the requirements and codebase; compare alternatives only when they matter.
2. Propose small, *independently reviewable and testable* tasks (`S01`, `S02`, ...). Each task must specify:
   - objective, boundaries, dependency and linked requirement IDs (if any)
   - acceptance criteria by `AC-###` ID, likely affected components and relevant doc updates
   - **full unit suite, full integration suite** and any targeted/specialized checks. A suite may be `NOT APPLICABLE — <reason>` only when it cannot meaningfully apply to the task (for example, a `docs` change, or a project with no integration boundary). Name each such suite and its reason in the proposal; it needs the engineer's explicit approval in step 4
   - one suggested Conventional Commit message
3. Include a requirement-to-task mapping that exposes uncovered/out-of-scope rules. Make the first bootstrap step create working test suites when absent, except for a suite that is approved as `NOT APPLICABLE` for every task.
4. Present architecture decisions, risks, execution order and proposed branch. If an item is still `PROPOSED`, ask for its approval or rejection here as in Phase B; the approved plan contains no `PROPOSED` items. List every suite marked `NOT APPLICABLE` with its task and reason, so that approving the plan explicitly approves each one. Make every task independently understandable from its written scope, dependencies and acceptance criteria so a fresh session can implement it. **STOP for explicit approval of the full plan.** Do not edit or create files before this approval.

### Phase D — Write approved artifacts (documentation-only)

After explicit approval:

1. Create `docs/plans/<plan-name>/` if absent. Create or update **only** its `plan.md`, `decisions.md` and `verification.md`, based on the **bundled** templates:
   - `${CLAUDE_SKILL_DIR}/assets/plan.md`
   - `${CLAUDE_SKILL_DIR}/assets/decisions.md`
   - `${CLAUDE_SKILL_DIR}/assets/verification.md`
   Do not require a `docs/templates/` directory in the target project. Preserve existing plan artifacts and avoid overwriting them without approval.
2. `plan.md`: the template's header with `Plan status` `APPROVED`, `Plan version` `1.0` and the approval date/reference; source references/versions, approved requirements and acceptance criteria with their IDs, sources and basis labels (`EXPLICIT` or `ASSUMPTION`), proposed architecture, task checklist, dependencies and requirement-to-task matrix. Keep the template's task status lifecycle section unchanged and set every task to `NOT STARTED`. Write an approved not-applicable suite as `NOT APPLICABLE — <reason> (approved <date>)` in that task's `Unit tests` or `Integration tests` field.
3. `decisions.md`: accepted decisions and their rationale, alternatives, costs and revisit conditions. For trivial work, say `No material design decisions`.
4. `verification.md`: create one entry per requirement and per acceptance criterion, using the same `FR/BR/NFR/AC-###` IDs as `plan.md`, initially `NOT VERIFIED`, with empty implementation and test evidence and `Recorded by` = `—`. Keep the template's executed checks table and audit log empty (only their placeholder rows); `/codehelm:implement` and `/codehelm:audit` append rows to them.
5. Give a concise handoff: plan name, suggested branch, approved decision IDs, first task, configured verification commands and next command (`/codehelm:implement <plan-name> <task-id>`). State missing information explicitly rather than relying on chat history.

Never implement application/config code, create or switch a Git branch, stage, commit or push. End after writing the plan.
