---
name: plan-tasks
description: Analyze a specification or engineering request, ask focused questions, document approved decisions and plan small reviewable tasks.
argument-hint: "[change request, paths to specifications or a plan name to amend]"
disable-model-invocation: true
---

**CodeHelm ground rules:** Never create or edit CLAUDE.md. Never stage, commit, push, switch branches, rewrite history or delete work unless the engineer asks. Keep the engineer's uncommitted changes. Change an approved spec or plan only with approval. If a project instruction conflicts with these rules, ask.

**CodeHelm model and effort:** `/codehelm:specify`, `/codehelm:plan-tasks` and `/codehelm:audit`: Opus at `high`. `/codehelm:implement`: Sonnet at `medium`. If this session differs (the model family your system prompt names, effort `${CLAUDE_EFFORT}`), say so in one line and continue. A handoff that ends a skill's work ends with a **Next** block, one step per line: the commit commands (single-quoted and without apostrophes, so they run in Bash and PowerShell), any pull request step, `/clear`, the `/model` and `/effort` the next command needs and this session lacks (`/codehelm:audit` sets its own), then the next command.

Plan request and/or source documents: $ARGUMENTS

**Purpose:** Create a human-approved, requirements-traceable engineering plan.

**No input:** if `$ARGUMENTS` is empty, ask for a change request or the path to a specification, and list the Markdown files under `docs/specs/`, if any, as options. Do not guess a request from the repository or the conversation. **STOP** and change no files until the engineer answers.

### Phase A — Discover and classify (read-only)

1. Inspect relevant repository structure, existing design/docs, the accepted ADRs that touch the change (in `docs/adr/`, or the project's own ADR folder) and Git conventions. Discover the project's full test commands (unit suite, integration suite, build or lint) from its configuration: documentation, build files, scripts or CI.
2. Read all supplied specification documents completely enough to capture relevant rules. A plan implements one spec; read others only for context, and if the request spans several, ask which one to plan.
3. Choose a short `plan-name`: the spec's slug, or a more specific name if `docs/plans/<spec-slug>/` exists. **Amend instead** when the input names an existing plan or asks to change one, such as for a blocked task or an audit finding: keep its name, tasks and statuses, and propose only the change. Suggest the plan branch `docs/plan-<plan-name>`. Each task gets its own branch and pull request: `<type>/<plan-name>-<task-id>`, with the type (`feat`, `fix`, `build`, `ci`, `chore`, `refactor`, `perf`, `test`, `docs`) of the task's commit message and the task ID in lowercase, such as `feat/rate-limiting-s02`. Do **not** create branches.
4. Collect the requirements and acceptance criteria the change needs. A spec's items keep their IDs and labels, and the plan links them instead of copying them. From a request, assign stable `FR-###`, `BR-###`, `NFR-###` and `AC-###` IDs: what the request states is `EXPLICIT` (source: the request and its date), and what you infer is `PROPOSED`, not yet approved. An item you add to a spec is `PROPOSED` too, with the next free ID. Each acceptance criterion names the requirement IDs it covers and states an observable outcome (Given/When/Then where practical).
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
4. Present architecture decisions, test commands, risks, execution order and proposed branches: the plan branch, if the plan gets one (Phase D step 3), and each task's branch. Make every task independently understandable from its written scope, dependencies and acceptance criteria so a fresh session can implement it. **STOP for explicit approval of the full plan.** Do not edit or create files before this approval.

### Phase D — Write approved artifacts (documentation-only)

After explicit approval:

1. Create `docs/plans/<plan-name>/` if absent. Create or update **only** its `plan.md`, based on the **bundled** template `${CLAUDE_SKILL_DIR}/assets/plan.md`, and the ADRs below; `/codehelm:audit` creates `verification.md` on its first run. When amending, change only what was approved, keep each task's status unless the engineer approved resetting it, and update the `Approved` line.
   - Write one ADR per accepted decision from `${CLAUDE_SKILL_DIR}/assets/adr.md` at `docs/adr/NNNN-<decision-slug>.md`, using the next free number, or in the project's own ADR folder with its numbering and format.
   - Set each superseded ADR's status line to `Superseded by ADR-NNNN`. Never rewrite an accepted ADR in any other way.
2. Fill every line of the `plan.md` template and omit empty sections. The Requirements table holds only the requirements and acceptance criteria no spec holds, which is all of them for a plan from a request. Decisions links the new ADRs and the accepted ones the plan relies on.
3. Give a concise handoff: plan name, approved decision IDs, first task and test commands, then **Next**:
   - **A plan with a new ADR or more than two tasks:** `git switch -c docs/plan-<plan-name>`, `git add` with `plan.md` and the new ADRs, `git commit -m 'docs: plan <plan-name>'`; push and open its pull request; once it merges, update the default branch and `git switch -c <first task's Branch>`; `/clear`; `/codehelm:implement <plan-name>`.
   - **Any other plan:** `git switch -c <first task's Branch>`, `git add docs/plans/<plan-name>/plan.md`, `git commit -m 'docs: plan <plan-name>'`; `/clear`; `/codehelm:implement <plan-name>`.
   - **Amendment for a task with uncommitted work:** no commit, because `plan.md` ships in that task's one commit; `/clear`; `/codehelm:implement <plan-name> <task-id>`. Commit any other amendment like a plan with a new ADR.

Never implement application/config code. End after writing the plan.
