---
name: specify
description: Turn an incomplete business brief or requirements document into a human-approved, source-traceable specification of one business feature without designing architecture or implementation tasks.
argument-hint: "[problem description or paths to source documents]"
disable-model-invocation: true
---

**CodeHelm ground rules:** Never create or edit CLAUDE.md. Never stage, commit, push, switch branches, rewrite history or delete work unless the engineer asks. Keep the engineer's uncommitted changes. Change an approved spec or plan only with approval. If a project instruction conflicts with these rules, ask.

**CodeHelm model and effort:** `/codehelm:specify`, `/codehelm:plan-tasks` and `/codehelm:audit`: Opus at `high`. `/codehelm:implement`: Sonnet at `medium`. If this session differs (the model family your system prompt names, effort `${CLAUDE_EFFORT}`), say so in one line and continue. A handoff names the `/model` and `/effort` the next command needs and this session lacks; `/codehelm:audit` sets its own.

Input: $ARGUMENTS

**Purpose:** Convert a problem description into a reviewed statement of **what** the system must do. Use this skill only when a specification is needed; for well-defined engineering tasks, `/codehelm:plan-tasks` can work directly from the request. Do **not** plan implementation, choose the architecture or write code.

**No input:** if `$ARGUMENTS` is empty, ask for the business brief, as text or a file path. Do not guess a source from the repository or the conversation. **STOP** and change no files until the engineer answers.

### Phase A — Discover (read-only)

1. Read the supplied brief, as text or source documents, and the bundled template at `${CLAUDE_SKILL_DIR}/assets/specification.md`. If any referenced source is inaccessible, report it; do not guess.
2. Inspect only relevant existing domain documentation, including the specs under `docs/specs/`, to avoid contradicting an already agreed contract. If a spec already covers the brief's feature, propose updating it (Phase D step 1). Do not scan the full codebase unless an existing behavior must be clarified.
3. Name the business features the brief covers, such as capture and refunds or webhooks delivery, never technical layers such as the API or the database. Extract the problem, actors/external systems, goals, stated scope, explicit functional requirements, business invariants, quality constraints, acceptance examples and delivery restrictions.
4. Keep **three categories distinct**: `EXPLICIT` (supported by a named source), `ASSUMPTION` (user-approved), `PROPOSED` (not yet approved). Preserve requirement IDs in source documents and assign stable `FR-###`, `BR-###`, `NFR-###`, `AC-###` IDs where needed; cite the source path + heading/page in the draft, or `§ Source brief` for a brief given as chat text.
5. Detect conflicts, missing state transitions, unclear error semantics, retries, concurrency rules, authorization boundaries and missing scope limits **only where relevant**. Never invent business rules, numeric SLA targets, permissions or features just to fill a template.

### Phase B — Clarify (mandatory stop)

1. Briefly summarize the problem and what is explicitly known. If the brief covers several features, propose one spec per feature, each with a slug and a one-line scope, and ask which one to specify in this run, so that each spec stays small enough to review.
2. Ask at most five numbered **material business-behavior questions** per round, most important first, each with options, their consequences where helpful and your recommended answer. Separate questions for the stakeholders from reasonable assumptions the engineer may approve when stakeholders are unavailable.
3. **Do not silently decide.** Note which unanswered points are blockers versus manageable, recorded assumptions.
4. **STOP for the engineer's answers.** Iterate only where material ambiguity remains. If there is no stakeholder available, request approval for clearly labeled assumptions rather than claiming they were confirmed externally.

### Phase C — Draft and review (no file edits)

1. Prepare a concise specification: fill the bundled `${CLAUDE_SKILL_DIR}/assets/specification.md` template and omit empty sections.
2. Give each requirement an ID and **traceable provenance**: source reference, user-approved assumption or visibly unapproved proposal. Keep acceptance scenarios tied to requirement IDs and express observable Given/When/Then outcomes.
3. Keep the specification technology-neutral unless the source imposes explicit technology constraints. **Do not** propose repositories/classes, database tables, locking techniques, endpoint designs, task breakdowns, commits or a detailed test implementation strategy.
4. Show contradictions, missing answers and scope risks clearly; never represent proposed behavior as a confirmed requirement. Ask for explicit approval of the complete draft and of each remaining `PROPOSED` item: an approved item becomes `ASSUMPTION` with an `A-###` entry, and an unapproved item becomes a `Q-###` open question.
5. **STOP for approval before writing any files.** If the user requests changes, revise the draft and seek approval again.

### Phase D — Save the approved specification (documentation only)

Only after explicit approval:

1. Suggest a stable slug that names the feature in kebab-case business terms, such as `capture-and-refunds`, never a technical layer. Create `docs/specs/` if needed and save to `docs/specs/<spec-slug>.md`, using the bundled template. Do not overwrite an existing specification without express approval; propose an update and highlight differences instead.
2. Fill the template's `Approved` line with the approval date and the source path. For a brief given as chat text, quote it unchanged under `## Source brief` and write `Source: § Source brief`, because the conversation is gone after `/clear`. Include source references, decisions on ambiguous business behavior, approved assumptions and unresolved questions or limitations. The saved specification contains no `PROPOSED` items: every requirement is `EXPLICIT` or `ASSUMPTION`, and anything unapproved is a `Q-###` open question. Preserve the original brief and source documents unchanged and respect confidentiality restrictions.
3. Confirm the file reflects the approved draft and report the saved path plus any unresolved issues. List the features left for later `/codehelm:specify` runs. Suggest the next steps in order: create the branch `docs/specify-<spec-slug>`, commit the specification on it (`docs: specify <spec-slug>`) and open its pull request; after it merges, update the default branch; run `/clear`, then `/codehelm:plan-tasks docs/specs/<spec-slug>.md`.
