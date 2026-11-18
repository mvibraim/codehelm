---
name: specify
description: Turn an incomplete business brief or requirements document into a human-approved, source-traceable project specification without designing architecture or implementation tasks.
argument-hint: "[problem description or paths to source documents]"
disable-model-invocation: true
---

Input: $ARGUMENTS

**Purpose:** Convert a problem description into a reviewed statement of **what** the system must do. Use this skill only when a specification is needed; for well-defined engineering tasks, `/plan-tasks` can work directly from the request. Do **not** plan implementation, choose the architecture or write code.

### Phase A — Discover (read-only)

1. Read `CLAUDE.md`, the supplied business brief/source documents and `docs/templates/specification.md`. If the input is a file path, read the file; if it is text, treat that text as the source. If any referenced source is inaccessible, report it; do not guess.
2. Inspect only relevant existing domain documentation to avoid contradicting an already agreed contract. Do not scan the full codebase unless an existing behavior must be clarified.
3. Extract the problem, actors/external systems, goals, stated scope, explicit functional requirements, business invariants, quality constraints, acceptance examples and delivery restrictions.
4. Keep **three categories distinct**: `EXPLICIT` (supported by a named source), `ASSUMPTION` (user-approved), `PROPOSED` (not yet approved). Preserve requirement IDs in source documents and assign stable `FR-###`, `BR-###`, `NFR-###`, `AC-###` IDs where needed; include the source path + heading/page or conversation reference in the draft.
5. Detect conflicts, missing state transitions, unclear error semantics, retries, concurrency rules, authorization boundaries and missing scope limits **only where relevant**. Never invent business rules, numeric SLA targets, permissions or features just to fill a template.
6. Inspect `git status --short` before proposing any file changes. Preserve existing or uncommitted documents.

### Phase B — Clarify (mandatory stop)

1. Briefly summarize the problem and what is explicitly known.
2. Ask a small, prioritized set of **material business-behavior questions**, grouping related questions. Separate questions for the stakeholders from reasonable assumptions the engineer may approve when stakeholders are unavailable.
3. Offer options and consequences where helpful, but **do not silently decide**. Note which unanswered points are blockers versus manageable, recorded assumptions.
4. **STOP for the engineer's answers.** Iterate only where material ambiguity remains. If there is no stakeholder available, request approval for clearly labeled assumptions rather than claiming they were confirmed externally.

### Phase C — Draft and review (no file edits)

1. Prepare a concise specification following `docs/templates/specification.md`, using only relevant sections. Include: business problem and goals, actors, scope/out-of-scope, functional requirements, business rules, nonfunctional expectations, acceptance scenarios, fixed constraints, approved assumptions and unresolved questions.
2. Give each requirement an ID and **traceable provenance**: source reference, user-approved assumption or visibly unapproved proposal. Keep acceptance scenarios tied to requirement IDs and express observable Given/When/Then outcomes.
3. Keep the specification technology-neutral unless the source imposes explicit technology constraints. **Do not** propose repositories/classes, database tables, locking techniques, endpoint designs, task breakdowns, commits or a detailed test implementation strategy.
4. Show contradictions, missing answers and scope risks clearly; never represent proposed behavior as a confirmed requirement. Ask for explicit approval of the complete draft and any remaining assumptions.
5. **STOP for approval before writing any files.** If the user requests changes, revise the draft and seek approval again.

### Phase D — Save the approved specification (documentation only)

Only after explicit approval:

1. Suggest a stable, descriptive slug. Save to `docs/specifications/<project-slug>.md`, using the template. Do not overwrite an existing specification without express approval; propose an update and highlight differences instead.
2. Include status/version/date, source references, decisions on ambiguous business behavior, approved assumptions and unresolved questions or limitations. Preserve the original assignment unchanged and respect confidentiality restrictions.
3. Confirm the file reflects the approved draft and report the saved path plus any unresolved issues. Suggest the next command: `/plan-tasks docs/specifications/<project-slug>.md`.
4. Do **not** create implementation tasks, choose an architecture, modify application/configuration code, stage, commit, push or switch/create branches.

**Context:** The approved Markdown file is the durable handoff to `/plan-tasks`; do not rely on the prior chat as the sole record of requirements.
