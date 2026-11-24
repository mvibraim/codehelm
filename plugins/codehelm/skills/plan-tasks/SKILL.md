---
name: plan-tasks
description: Analyze a specification or engineering request, ask focused questions, document approved decisions and plan small reviewable tasks.
argument-hint: "[change request, paths to specifications or a plan name to amend]"
disable-model-invocation: true
---

**CodeHelm ground rules:** Never create or edit CLAUDE.md. Never stage, commit, push, switch branches, rewrite history or delete work unless the engineer asks, and keep their uncommitted changes. Change an approved spec or plan only with approval. If a project instruction conflicts with these rules, ask.

**CodeHelm model and effort:** `/codehelm:specify`, `/codehelm:plan-tasks` and `/codehelm:audit` run on Opus at `high`, `/codehelm:implement` on Sonnet at `medium`. If this session differs (your system prompt's model family, effort `${CLAUDE_EFFORT}`), say so in one line and continue. A handoff that ends a skill's work ends with a **Next** block, one step per line: commit commands (single-quoted, no apostrophes, so they run in Bash and PowerShell), any pull request step, `/clear`, the next command's `/model` and `/effort` (`/codehelm:audit` sets its own), then the next command.

Plan request and/or source documents: $ARGUMENTS

**No input:** ask for a change request or a spec path, listing the Markdown files under `docs/specs/`. Don't guess one from the repository or the conversation, and change no files until the engineer answers.

### Phase A — Discover (read-only)

1. Inspect the relevant code and docs, the accepted ADRs the change touches (in `docs/adr/` or the project's ADR folder) and Git conventions. Find the full test commands (unit suite, integration suite, build or lint) in the project's docs, build files, scripts or CI.
2. Read the supplied spec. A plan implements one spec; if the request spans several, ask which.
3. Name the plan after the spec's slug, or more specifically if `docs/plans/<spec-slug>/` exists. **Amend instead** when the input names an existing plan or asks to change one, such as for a blocked task or an audit finding: keep its name, tasks and statuses and propose only the change. Branches are `docs/plan-<plan-name>` for the plan and `<type>/<plan-name>-<task-id>-<task-name>` per task, `<task-name>` being the task title in a few kebab-case words, such as `feat/rate-limiting-s02-json-429`.
4. Collect the requirements and acceptance criteria the change needs, linking a spec's items by ID instead of copying them. For a request, assign `FR-###`, `BR-###`, `NFR-###` and `AC-###` IDs: what it states is `EXPLICIT` (source: the request and its date), what you infer is `PROPOSED`. An item you add to a spec is `PROPOSED`, with the next free ID. Each acceptance criterion names the requirements it covers and an observable outcome, Given/When/Then where practical.
5. Find scope boundaries, contradictions, unknowns and edge cases. Drift checks need a clean commit to record, so before approval ask the engineer to fix: no Git repository or no commit; a `Sources` file modified or untracked in `git status --short`; a spec committed only on an unmerged branch (merge it and update the default branch, since a squash merge can drop the recorded commit).

### Phase B — Clarify

1. Ask at most five numbered questions per round, most important first, each with options and your recommendation.
2. Separate requirements from assumptions and engineering choices. Ask the engineer to approve or reject each `PROPOSED` item, once, here: approved becomes `ASSUMPTION`, rejected goes out of scope.
3. **STOP and wait** for answers. Repeat only for essential ambiguity.

### Phase C — Propose (read-only)

1. Recommend the simplest design that meets the requirements; compare alternatives only when it matters. No speculative features and no wrappers around framework features. Propose an ADR only for a decision later tasks must respect or that is costly to reverse, and name smaller choices in the task's scope. Changing an accepted decision needs a new ADR that supersedes it.
2. Propose tasks `S01`, `S02`, ...: each one concern, a diff reviewable in about ten minutes and, except a setup task, a vertical slice that changes observable behavior. Each ships as its own pull request after its dependencies merge. Give each its objective and scope, dependencies, covered requirement and `AC-###` IDs, docs to update, **extra checks** beyond the test commands (such as a container smoke test or a migration test, or none), its branch and a Conventional Commit message. Over about eight tasks, propose splitting the spec with `/codehelm:specify` instead.
3. Propose the **Test commands**: the full unit and integration suites and the build or lint command. Every task runs them all. If a unit or integration suite is missing, the first task sets it up and every other task depends on it, unless the engineer approves "no integration tests" with a reason for the whole plan. List the requirements no task covers and those out of scope.
4. Present decisions, test commands, risks, order and branches, each task understandable on its own by a fresh session. **STOP for explicit approval of the full plan** before writing any file.

### Phase D — Write (documentation only)

1. Write `docs/plans/<plan-name>/plan.md` from `${CLAUDE_SKILL_DIR}/assets/plan.md`, filling every line and omitting empty sections. The `Approved` line records `git rev-parse HEAD`. The Requirements table holds only items no spec holds; Decisions links new and relied-on ADRs. `/codehelm:audit` creates `verification.md`. When amending, change only what was approved, keep statuses unless the engineer approved a reset, and update the `Approved` line.
2. Write one ADR per accepted decision from `${CLAUDE_SKILL_DIR}/assets/adr.md` at `docs/adr/NNNN-<decision-slug>.md` with the next free number, or in the project's ADR folder and format. Change an accepted ADR only to set its status to `Superseded by ADR-NNNN`.
3. Hand off the plan name, decision IDs, first task and test commands, then **Next**, ending with `/clear`, `/model sonnet`, `/effort medium`, `/codehelm:implement <plan-name>`:
   - **A new ADR or more than two tasks:** `git switch -c docs/plan-<plan-name>`, `git add` with `plan.md` and the new ADRs, `git commit -m 'docs: plan <plan-name>'`; push and open its pull request; once it merges, update the default branch and `git switch -c <first task's Branch>`.
   - **Any other plan:** `git switch -c <first task's Branch>`, `git add docs/plans/<plan-name>/plan.md`, `git commit -m 'docs: plan <plan-name>'`.
   - **Amendment for a task with uncommitted work:** no commit, because `plan.md` ships in that task's commit; the command adds `<task-id>`. Commit other amendments like a plan with a new ADR.
