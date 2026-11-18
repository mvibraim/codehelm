---
name: plan-tasks
description: Analyze a specification or engineering request, ask focused questions, document approved decisions and plan small reviewable tasks.
argument-hint: "[change request or paths to specifications]"
disable-model-invocation: true
---

**CodeHelm portability and control:** This skill is self-contained. Respect an existing project `CLAUDE.md` when present; never create, copy, replace or merge one as a prerequisite. Follow existing repository conventions and preserve the engineer's uncommitted work. Never stage, commit, push, create/switch branches, rewrite history or perform destructive changes without explicit approval for that action. Project specifications and approved decisions are durable sources of truth; do not silently change them. If project instructions conflict with a CodeHelm safeguard, stop and ask the engineer to resolve the conflict. If Git is not initialized, report that fact and never initialize it without approval.

Plan request and/or source documents: $ARGUMENTS

**Purpose:** Create a human-approved, requirements-traceable engineering plan. This skill is task-neutral: it applies to features, fixes, Docker, CI, observability, refactors and other changes.

### Phase A — Discover and classify (read-only)

1. Read the project's `CLAUDE.md` **if present**, and respect its project-specific instructions. CodeHelm does not supply or require that file; when absent, work from this skill and the repository's existing conventions. Inspect relevant repository structure, existing design/docs, build/test commands and Git conventions. Check `git status --short` when Git is initialized; otherwise note the absence of Git and do not initialize it. Do not modify anything yet. **Prefer scoped searches and relevant file sections over loading the whole repository.**
2. Read all supplied specification documents completely enough to capture relevant rules. If a referenced file cannot be read, report that limitation instead of guessing. Do not alter original specifications.
3. Choose a short `plan-name` and suggest a change type (`feat`, `fix`, `build`, `ci`, `chore`, `refactor`, `perf`, `test`, `docs`) and branch name. Do **not** create the branch.
4. Extract explicit functional requirements, business rules, nonfunctional constraints, error behavior and acceptance criteria. Preserve existing IDs; otherwise assign stable `FR-###`, `BR-###`, `NFR-###` IDs. Cite source file + section/page where practical. If no specification exists, derive a provisional requirements list from the request and label it as proposed.
5. Identify scope boundaries, contradictions, unknowns, dependencies, edge cases and source versions. Where practical record `git rev-parse HEAD` and source file SHA-256 fingerprints so later steps can detect specification drift.

### Phase B — Clarify (mandatory gate)

1. Ask a **small, prioritized group** of material questions; provide reasonable alternatives and your tentative recommendation.
2. Explicitly distinguish requirements from assumptions and engineering choices.
3. **STOP and wait** for answers. Repeat only when essential ambiguities remain. Do not produce authoritative plan files yet.

### Phase C — Propose design and execution plan (read-only)

1. Recommend the simplest design consistent with the requirements and codebase; compare alternatives only when they matter.
2. Propose small, *independently reviewable and testable* tasks (`S01`, `S02`, ...). Each task must specify:
   - objective, boundaries, dependency and linked requirement IDs (if any)
   - acceptance criteria, likely affected components and relevant doc updates
   - **full unit suite, full integration suite** and any targeted/specialized checks
   - one suggested Conventional Commit message
3. Include a requirement-to-task mapping that exposes uncovered/out-of-scope rules. Make the first bootstrap step create working test suites when absent.
4. Present architecture decisions, risks, execution order and proposed branch. Make every task independently understandable from its written scope, dependencies and acceptance criteria so a fresh session can implement it. **STOP for explicit approval of the full plan.** Do not edit or create files before this approval.

### Phase D — Write approved artifacts (documentation-only)

After explicit approval:

1. Create `docs/plans/<plan-name>/` if absent. Create or update **only** its `plan.md`, `decisions.md` and `verification.md`, based on the **bundled** templates:
   - `${CLAUDE_SKILL_DIR}/assets/plan.md`
   - `${CLAUDE_SKILL_DIR}/assets/decisions.md`
   - `${CLAUDE_SKILL_DIR}/assets/verification.md`
   Do not require a `docs/templates/` directory in the target project. Preserve existing plan artifacts and avoid overwriting them without approval.
2. `plan.md`: source references/versions, approved requirements, proposed architecture, acceptance criteria, task checklist, dependencies and requirement-to-task matrix.
3. `decisions.md`: accepted decisions and their rationale, alternatives, costs and revisit conditions. For trivial work, say `No material design decisions`.
4. `verification.md`: create one entry per requirement/acceptance criterion, initially `NOT VERIFIED`, with empty implementation and test evidence.
5. Give a concise handoff: plan name, suggested branch, approved decision IDs, first task, configured verification commands and next command (`/implement <plan-name> <task-id>`). State missing information explicitly rather than relying on chat history.

Never implement application/config code, create or switch a Git branch, stage, commit or push. End after writing the plan.
