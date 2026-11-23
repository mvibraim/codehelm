---
name: plan-tasks
description: Analyze a specification or engineering request, ask focused questions, document approved decisions and plan small reviewable tasks.
argument-hint: "[change request, paths to specifications or a plan name to amend]"
disable-model-invocation: true
---

**CodeHelm ground rules:** Never create or edit CLAUDE.md. Never stage, commit, push, switch branches, rewrite history or delete work unless the engineer asks. Keep the engineer's uncommitted changes. Change an approved spec or plan only with approval. If a project instruction conflicts with these rules, ask.

**CodeHelm model and effort:** `/codehelm:specify`, `/codehelm:plan-tasks` and `/codehelm:audit`: Opus at `high`. `/codehelm:implement`: Sonnet at `medium`. If this session differs (the model family your system prompt names, effort `${CLAUDE_EFFORT}`), say so in one line and continue. A handoff names the `/model` and `/effort` the next command needs and this session lacks; `/codehelm:audit` sets its own.

Plan request and/or source documents: $ARGUMENTS

**Purpose:** Create a human-approved, requirements-traceable engineering plan.

**No input:** if `$ARGUMENTS` is empty, ask for a change request or the path to a specification, and list the Markdown files under `docs/specs/`, if any, as options. Do not guess a request from the repository or the conversation. **STOP** and change no files until the engineer answers.

### Phase A — Discover and classify (read-only)

1. Inspect relevant repository structure, existing design/docs, the accepted ADRs that touch the change (in `docs/adr/`, or the project's own ADR folder) and Git conventions. Discover the project's full test commands (unit suite, integration suite, build or lint) from its configuration: documentation, build files, scripts or CI.
2. Read all supplied specification documents completely enough to capture relevant rules. A plan implements one spec; read others only for context, and if the request spans several, ask which one to plan.
3. Choose a short `plan-name`: the spec's slug, or a more specific name if `docs/plans/<spec-slug>/` exists. **Amend instead** when the input names an existing plan or asks to change one, such as for a blocked task or an audit finding: keep its name, tasks and statuses, and propose only the change. Suggest the plan branch `docs/plan-<plan-name>`. Each task gets its own branch and pull request: `<type>/<plan-name>-<task-id>`, with the type (`feat`, `fix`, `build`, `ci`, `chore`, `refactor`, `perf`, `test`, `docs`) of the task's commit message and the task ID in lowercase, such as `feat/rate-limiting-s02`. Do **not** create branches.
4. Extract explicit functional requirements, business rules, nonfunctional constraints, error behavior and acceptance criteria. Preserve existing IDs and provenance labels unchanged; otherwise assign stable `FR-###`, `BR-###`, `NFR-###` and `AC-###` IDs. Cite source file + section/page where practical. Give every requirement and acceptance criterion exactly one basis label:
   - `EXPLICIT`: stated in a named source (a specification section, or the engineer's request with its date).
   - `ASSUMPTION`: not stated in a source, but approved by the engineer.
   - `PROPOSED`: inferred or suggested by you and not yet approved.
   If no specification exists, derive the list from the request: what the request states directly is `EXPLICIT` (source: the request), and everything you infer is `PROPOSED`. Each acceptance criterion names the requirement IDs it covers and states an observable outcome (Given/When/Then where practical).
5. Identify scope boundaries, contradictions, unknowns, dependencies and edge cases. The plan records the commit at approval (`git rev-parse HEAD`), and later skills detect source drift with `git diff <sha> -- <source>`. If the project is not a Git repository, has no commit yet, or `git status --short` shows a file the plan's `Sources` line names as modified or untracked, ask the engineer to set up Git or commit before approving the plan, because drift can't be detected otherwise. If the spec is committed only on an unmerged branch, ask the engineer to merge its pull request and update the default branch first, because a squash merge can drop the commit the plan records.

### Phase B — Clarify (mandatory gate)

1. Ask at most five numbered material questions per round, most important first, each with options and your recommended answer.
2. Explicitly distinguish requirements from assumptions and engineering choices. List every `PROPOSED` requirement or acceptance criterion and ask the engineer to approve or reject each. Ask this once, here: an approved item becomes `ASSUMPTION`, a rejected item moves out of scope, and the plan you propose in Phase C contains no `PROPOSED` items.
3. **STOP and wait** for answers. Repeat only when essential ambiguities remain.

### Phase C — Propose design and execution plan (read-only)

1. Recommend the simplest design consistent with the requirements and codebase; compare alternatives only when they matter. Plan only what the requirements need: no speculative or "might need" features, and no wrappers around framework or library features. Every new dependency or abstraction needs its own ADR, and changing an accepted decision needs a new ADR that supersedes it.
2. Propose small, *independently reviewable and testable* tasks (`S01`, `S02`, ...). Each task covers one concern, produces a diff a reviewer can read in about ten minutes and, apart from a setup task (step 3), is a vertical slice that changes observable behavior rather than one layer of several features. Each task ships as its own pull request, started from the default branch after its dependencies' pull requests merge. Each task must specify:
   - objective, boundaries, dependency and linked requirement IDs (if any)
   - acceptance criteria by `AC-###` ID, likely affected components and relevant doc updates
   - **extra checks**: only the checks this task needs beyond the plan's test commands (for example, a container build and smoke test, a workflow lint or a migration test), or none
   - its branch (Phase A step 3) and one suggested Conventional Commit message

   More than about eight tasks is too big for one plan to review and deliver task by task: say so and propose splitting the spec with `/codehelm:specify` before approval.
3. Propose the plan's **Test commands**: the full unit and integration suites and the build or lint command found in Phase A step 1. Every task runs all of them, so approving the plan approves them. If the project lacks a unit or integration suite, make the first task set it up and every other task depend on it; the only alternative is that the engineer approves "no integration tests", with a reason, once for the whole plan. In the proposal, list the requirements and acceptance criteria that no task covers and those out of scope.
4. **Lite path:** if the change needs only one or two tasks and involves no real design choice, propose only the goal and scope, the test commands and the tasks, and state each task's acceptance criteria in its **Covers** line, for example `AC-001` (`EXPLICIT`): Given ... when ... then .... Skip the requirements tables, decisions and risks.
5. Present architecture decisions, test commands, risks, execution order and proposed branches: the plan branch and each task's branch (on the lite path, only the goal, test commands and tasks with their branches). Make every task independently understandable from its written scope, dependencies and acceptance criteria so a fresh session can implement it. **STOP for explicit approval of the full plan.** Do not edit or create files before this approval.

### Phase D — Write approved artifacts (documentation-only)

After explicit approval:

1. Create `docs/plans/<plan-name>/` if absent. Create or update **only** its `plan.md`, based on the **bundled** template `${CLAUDE_SKILL_DIR}/assets/plan.md`, and the ADRs below; `/codehelm:audit` creates `verification.md` on its first run. When amending, change only what was approved, keep each task's status unless the engineer approved resetting it, and update the `Approved` line.
   - Write one ADR per accepted decision from `${CLAUDE_SKILL_DIR}/assets/adr.md` at `docs/adr/NNNN-<decision-slug>.md`, using the next free number, or in the project's own ADR folder with its numbering and format.
   - Set each superseded ADR's status line to `Superseded by ADR-NNNN`. Never rewrite an accepted ADR in any other way.
2. `plan.md`: the `Approved` line with the approval date and the commit at approval (`git rev-parse HEAD`); the sources; goal and scope; test commands; approved requirements and acceptance criteria with their IDs, sources and basis labels (`EXPLICIT` or `ASSUMPTION`); the Decisions section, linking the new ADRs and the accepted ones the plan relies on (for trivial work, write `No material design decisions`); the tasks, each set to `NOT STARTED` with its **Branch** line; and risks. When the engineer approved a plan without integration tests, write `Integration: none — <reason> (approved <date>)` in the test commands. Mark each test command the setup task creates with `(created by S01)`. On the lite path, write only the `Approved` and `Sources` lines, goal and scope, test commands and tasks, with each task's acceptance criteria in its **Covers** line, and omit the other sections.
3. Give a concise handoff: plan name, suggested branches, approved decision IDs, first task, test commands and the next steps in order. Say that each task gets its own pull request.
   - **Full plan:** create the branch `docs/plan-<plan-name>`, commit `plan.md` and the new ADRs together on it (`docs: plan <plan-name>`) and open its pull request; after it merges, update the default branch and create the first task's branch; run `/clear`, then `/codehelm:implement <plan-name>`.
   - **Lite plan:** create the first task's branch, commit `plan.md` on it (`docs: plan <plan-name>`), so that the plan ships in that task's pull request, run `/clear`, then `/codehelm:implement <plan-name>`.
   - **Amendment for a task with uncommitted work:** leave `plan.md` uncommitted so it ships in that task's one commit; run `/clear`, then `/codehelm:implement <plan-name> <task-id>`. Commit any other amendment like a full plan.

Never implement application/config code. End after writing the plan.
