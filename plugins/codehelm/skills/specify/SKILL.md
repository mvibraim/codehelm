---
name: specify
description: Turn an incomplete business brief or requirements document into a human-approved, source-traceable specification of one business feature without designing architecture or implementation tasks.
argument-hint: "[problem description or paths to source documents]"
disable-model-invocation: true
---

**CodeHelm ground rules:** Never create or edit CLAUDE.md. Never stage, commit, push, switch branches, rewrite history or delete work unless the engineer asks, and keep their uncommitted changes. Change an approved spec or plan only with approval. If a project instruction conflicts with these rules, ask.

**CodeHelm model and effort:** `/codehelm:specify`, `/codehelm:plan-tasks` and `/codehelm:audit` run on Opus at `high`, `/codehelm:implement` on Sonnet at `medium`. If this session differs (your system prompt's model family, effort `${CLAUDE_EFFORT}`), say so in one line and continue. A handoff that ends a skill's work ends with a **Next** block, one step per line: commit commands (single-quoted, no apostrophes, so they run in Bash and PowerShell), any pull request step, `/clear`, any `/model` or `/effort` the next command needs (`/codehelm:audit` sets its own), then the next command.

Input: $ARGUMENTS

**No input:** ask for the business brief, as text or a file path. Don't guess one from the repository or the conversation, and change no files until the engineer answers.

### Phase A — Discover (read-only)

1. Read the brief and `${CLAUDE_SKILL_DIR}/assets/specification.md`. Report any source you can't access.
2. Read the specs under `docs/specs/` and other relevant domain docs so you don't contradict them. If a spec already covers the feature, propose updating it and show the differences. Read code only to clarify existing behavior.
3. Name the business features the brief covers, such as capture and refunds, never technical layers such as the API. Extract the problem, actors, goals, scope, requirements, business rules, quality constraints, examples and delivery restrictions.
4. Label every item `EXPLICIT` (from a named source), `ASSUMPTION` (approved by the engineer) or `PROPOSED` (not yet approved). Keep the sources' IDs and assign `FR-###`, `BR-###`, `NFR-###` and `AC-###` where needed. Cite the source path and heading, or `§ Source brief` for a brief given in chat.
5. Where relevant, find conflicts, missing state transitions, unclear errors, retries, concurrency, authorization and scope limits. Never invent rules, SLA numbers, permissions or features to fill the template; write `TBD` for an unknown number.

### Phase B — Clarify

1. Summarize the problem and what is known, and propose the spec's slug in kebab-case business terms, such as `capture-and-refunds`. If the brief covers several features, propose one spec each, with a slug and a one-line scope, and ask which to specify now.
2. Ask at most five numbered business-behavior questions per round, most important first, each with options, consequences and your recommendation. Separate questions for stakeholders from assumptions the engineer may approve without them, and mark which open points block.
3. **STOP for the engineer's answers.** Repeat only for material ambiguity. Never present an assumption as confirmed by stakeholders.

### Phase C — Draft (no file edits)

1. Draft the spec from the template, omitting empty sections. Tie each acceptance scenario to requirement IDs with an observable Given/When/Then.
2. Stay technology-neutral unless the source imposes technology: no classes, tables, endpoints, locking, tasks, commits or test strategy.
3. Show contradictions, gaps and scope risks. Ask for approval of the draft and of each `PROPOSED` item: approved becomes `ASSUMPTION` with an `A-###` entry, unapproved becomes a `Q-###` question.
4. **STOP for approval before writing any file.** If the engineer asks for changes, revise and ask again.

### Phase D — Save (documentation only)

1. Save to `docs/specs/<spec-slug>.md`, with no `PROPOSED` items. Quote a brief given in chat unchanged under `## Source brief`, because the conversation is gone after `/clear`. Leave source documents unchanged.
2. Report the path, unresolved issues and the features left for later `/codehelm:specify` runs. **Next:** `git switch -c docs/specify-<spec-slug>`, `git add docs/specs/<spec-slug>.md`, `git commit -m 'docs: specify <spec-slug>'`; push and open its pull request; once it merges, update the default branch; `/clear`; `/codehelm:plan-tasks docs/specs/<spec-slug>.md`.
