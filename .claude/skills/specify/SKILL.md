---
name: specify
description: Turn an incomplete business brief or requirements document into a human-approved, source-traceable project specification without designing architecture or implementation tasks.
argument-hint: "[problem description or paths to source documents]"
disable-model-invocation: true
---

**CodeHelm portability and control:** This skill is self-contained. Respect an existing project `CLAUDE.md` when present; never create, copy, replace or merge one as a prerequisite. Follow existing repository conventions and preserve the engineer's uncommitted work. Never stage, commit, push, create/switch branches, rewrite history or perform destructive changes without explicit approval for that action. Project specifications and approved decisions are durable sources of truth; do not silently change them. If project instructions conflict with a CodeHelm safeguard, stop and ask the engineer to resolve the conflict. If Git is not initialized, report that fact and never initialize it without approval.

Input: $ARGUMENTS

**Purpose:** Convert a problem description into a reviewed statement of **what** the system must do. Use this skill only when a specification is needed; for well-defined engineering tasks, `/plan-tasks` can work directly from the request. Do **not** plan implementation, choose the architecture or write code.

**No input:** if `$ARGUMENTS` is empty, ask for the business brief, as text or a file path. Do not guess a source from the repository or the conversation. **STOP** and change no files until the engineer answers.

### Phase A — Discover (read-only)

1. Read the project's `CLAUDE.md` **if one exists**; it belongs to that project and is **not provided or overwritten by CodeHelm**. If absent, continue using this self-contained skill and the repository's actual conventions. Read the supplied business brief/source documents and the bundled template at `${CLAUDE_SKILL_DIR}/assets/specification.md`. If the input is a file path, read the file; if it is text, treat that text as the source. If any referenced source is inaccessible, report it; do not guess.
2. Inspect only relevant existing domain documentation to avoid contradicting an already agreed contract. Do not scan the full codebase unless an existing behavior must be clarified.
3. Extract the problem, actors/external systems, goals, stated scope, explicit functional requirements, business invariants, quality constraints, acceptance examples and delivery restrictions.
4. Keep **three categories distinct**: `EXPLICIT` (supported by a named source), `ASSUMPTION` (user-approved), `PROPOSED` (not yet approved). Preserve requirement IDs in source documents and assign stable `FR-###`, `BR-###`, `NFR-###`, `AC-###` IDs where needed; include the source path + heading/page or conversation reference in the draft.
5. Detect conflicts, missing state transitions, unclear error semantics, retries, concurrency rules, authorization boundaries and missing scope limits **only where relevant**. Never invent business rules, numeric SLA targets, permissions or features just to fill a template.
6. If Git is initialized, inspect `git status --short` before proposing file changes; otherwise note the absence of Git. Preserve existing or uncommitted documents.

### Phase B — Clarify (mandatory stop)

1. Briefly summarize the problem and what is explicitly known.
2. Ask a small, prioritized set of **material business-behavior questions**, grouping related questions. Separate questions for the stakeholders from reasonable assumptions the engineer may approve when stakeholders are unavailable.
3. Offer options and consequences where helpful, but **do not silently decide**. Note which unanswered points are blockers versus manageable, recorded assumptions.
4. **STOP for the engineer's answers.** Iterate only where material ambiguity remains. If there is no stakeholder available, request approval for clearly labeled assumptions rather than claiming they were confirmed externally.

### Phase C — Draft and review (no file edits)

1. Prepare a concise specification following the bundled `${CLAUDE_SKILL_DIR}/assets/specification.md` template, using only relevant sections. Include: business problem and goals, actors, scope/out-of-scope, functional requirements, business rules, nonfunctional expectations, acceptance scenarios, fixed constraints, approved assumptions and unresolved questions.
2. Give each requirement an ID and **traceable provenance**: source reference, user-approved assumption or visibly unapproved proposal. Keep acceptance scenarios tied to requirement IDs and express observable Given/When/Then outcomes.
3. Keep the specification technology-neutral unless the source imposes explicit technology constraints. **Do not** propose repositories/classes, database tables, locking techniques, endpoint designs, task breakdowns, commits or a detailed test implementation strategy.
4. Show contradictions, missing answers and scope risks clearly; never represent proposed behavior as a confirmed requirement. Ask for explicit approval of the complete draft and any remaining assumptions.
5. **STOP for approval before writing any files.** If the user requests changes, revise the draft and seek approval again.

### Phase D — Save the approved specification (documentation only)

Only after explicit approval:

1. Suggest a stable, descriptive slug. Create `docs/specifications/` if needed and save to `docs/specifications/<project-slug>.md`, using the bundled template. Do not overwrite an existing specification without express approval; propose an update and highlight differences instead.
2. Set the template's status to `Approved` with version, update date and approval reference (date and approver), and include source references, decisions on ambiguous business behavior, approved assumptions and unresolved questions or limitations. Preserve the original brief and source documents unchanged and respect confidentiality restrictions.
3. Confirm the file reflects the approved draft and report the saved path plus any unresolved issues. Suggest the next command: `/plan-tasks docs/specifications/<project-slug>.md`.
4. Do **not** create implementation tasks, choose an architecture, modify application/configuration code, stage, commit, push or switch/create branches.

**Context:** The approved Markdown file is the durable handoff to `/plan-tasks`; do not rely on the prior chat as the sole record of requirements.
