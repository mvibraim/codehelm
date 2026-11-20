# CodeHelm

<p align="center">
  <img
    src="plugins/codehelm/.claude-plugin/icon.png"
    alt="CodeHelm"
    width="300"
  />
</p>

<p align="center">
  <strong>Let Claude write the code. You steer the ship.</strong>
</p>

<p align="center">
  A lightweight human-in-the-loop workflow for Claude Code.
  <br />
  Recommended for engineers who love AI but refuse to surrender the steering wheel.
  <br />
  Specify → Plan → Implement → Audit: one tested and reviewable change at a time.
</p>

<br>

No endless specification documents. No mysterious AI-generated commits. No waking up to discover Claude refactored your entire application. Just clear requirements, documented decisions, automated tests and a human firmly holding the wheel.

**AI is your crew, not your captain.**

## The four skills

- **`/codehelm:specify` — Understand the destination.** Turn incomplete briefs into clear, approved requirements; optional when the task is already well-defined.
- **`/codehelm:plan-tasks` — Chart the course.** Decide how to build it and break the work into small reviewable tasks.
- **`/codehelm:implement` — Sail one leg at a time.** Implement a single approved task, verify it, then stop for human review.
- **`/codehelm:audit` — Check the navigation.** Compare specifications, decisions, code and test evidence.

## Structure

CodeHelm is a Claude Code plugin. This repository is also its marketplace.

```text
.claude-plugin/marketplace.json     # marketplace "codehelm"
plugins/codehelm/
  .claude-plugin/
    plugin.json                     # plugin "codehelm"
    icon.png                        # plugin icon
  skills/
    specify/
      SKILL.md
      assets/specification.md
    plan-tasks/
      SKILL.md
      assets/plan.md
    implement/SKILL.md
    audit/
      SKILL.md
      assets/verification.md
```

## Install

In a Claude Code session (v2.1.275 or later), run:

```text
/plugin install codehelm --marketplace mvibraim/codehelm
```

Claude Code adds the marketplace after you confirm it, then asks for an install scope. Choose **project scope** to enable CodeHelm for everyone working in the repository, or **user scope** to use it in all your projects. On older versions, add the marketplace first:

```text
/plugin marketplace add mvibraim/codehelm
/plugin install codehelm@codehelm
```

From a shell, `claude plugin install codehelm --marketplace mvibraim/codehelm --scope project` does the same without a session. With project scope, each collaborator still runs the install once per clone.

Plugin skills are namespaced, so CodeHelm's skills run as `/codehelm:specify`, `/codehelm:plan-tasks`, `/codehelm:implement` and `/codehelm:audit` and never overwrite or shadow your own skills. The bare form, such as `/implement`, also works when no other skill or command uses that name. This README always uses the namespaced form.

**Updating:** the plugin has a fixed `version`, so new releases reach you when the version changes. Run `claude plugin update codehelm@codehelm`, or turn on auto-update for the `codehelm` marketplace in `/plugin`.

## Skill usage: terminal, context and model

A **fresh Claude Code session** is more important than opening a new terminal. You can start a new `claude` session in a fresh terminal **or** run `/clear` in the existing Claude Code terminal, but only after decisions and progress are saved to Markdown. Use `/model` to choose a model; these are suggestions, not hard requirements or settings embedded in the skills.

| Skill | Fresh session? | Suggested model | Why |
| --- | --- | --- | --- |
| `/codehelm:specify` | Prefer a fresh session when starting from a new or long brief | **Opus** for ambiguous briefs; **Sonnet** for straightforward ones | Careful business-question discovery without unrelated implementation context |
| `/codehelm:plan-tasks` | Prefer a fresh session after the specification is approved | **Opus** for architecture/trade-offs; **Sonnet** for small changes | Build a plan from saved requirements rather than old conversation assumptions |
| `/codehelm:implement` | Prefer a fresh session **for each approved task** | **Sonnet** by default; **Opus** for difficult concurrency or debugging | Reduce context and keep each reviewable change narrowly scoped |
| `/codehelm:audit` | **Yes**, use an independent fresh session when practical | **Sonnet** for task audits; **Opus** for critical/full reviews | Review evidence skeptically without relying on the implementation discussion |

**Practical commands:** `/model opus` or `/model sonnet`, then invoke the skill. After a completed phase or task, use `/clear` (or exit and start `claude` again) **only after** recording decisions, test results and unfinished work. Use `/compact` instead if a task is still in progress and the context is long. You don't need a new terminal window for every skill and frequent model switching mid-task is unnecessary.

## 0. Specify (optional: for an incomplete business brief)

Store the original project brief unchanged in a markdown file, then run:

```text
/codehelm:specify docs/briefs/<brief>.md
```

Claude extracts only sourced requirements, identifies gaps, asks **business clarification questions** (at most five numbered questions per round, each with options and a recommended answer) and **stops**; when stakeholders are unavailable you approve clearly documented assumptions. Claude then drafts a specification without architecture or implementation tasks and **stops again for approval**. Anything you don't approve becomes an open question, so the saved specification contains no `PROPOSED` items. After approval it saves:

```text
docs/specifications/<spec-slug>.md
```

The saved file records its approval date and source, because it is written only after your approval. If you already have an approved specification, skip this phase.

## 1. Plan

```text
/codehelm:plan-tasks docs/specifications/<spec-slug>.md
```

Or without a formal specification:

```text
/codehelm:plan-tasks Add GitHub Actions for unit and integration tests
/codehelm:plan-tasks Add a Dockerfile with a smoke test
/codehelm:plan-tasks Fix duplicate booking creation
```

Claude reads sources, asks at most five numbered clarification questions per round, each with options and a recommended answer, **stops for your answers**, proposes architecture and small tasks, **stops for approval**, then writes:

```text
docs/plans/<plan-name>/plan.md
```

The plan holds the goal, test commands, requirements, decisions and tasks, and records the commit it was approved at, so later skills can detect changes to its sources with `git diff`. A change with one or two tasks and no real design choice gets a lite plan: goal, test commands and tasks only, with each task's acceptance criteria written in the task. Commit the specification before approving the plan. It suggests a branch; **you** create/switch branches and commit the planning docs when ready. Respect any confidentiality restrictions on the source documents before committing them.

### Example of small, reviewable tasks

For illustration, a plan for an **Inventory Reservation API** might contain tasks like these. This is **not** a fixed task list: `/codehelm:plan-tasks` must derive the actual tasks from the approved requirements and decisions.

| Task | Small, reviewable change | Example verification |
| --- | --- | --- |
| `S01` | Establish the application, persistence setup, and working test infrastructure | Build succeeds; unit and integration suites run |
| `S02` | Expose a read-only product availability endpoint | Availability unit tests and API/database integration tests |

Each task should have acceptance criteria (`AC-###`), links to applicable requirements, and a suggested commit message. The plan keeps the specification's requirement and acceptance criteria IDs and its `EXPLICIT` / `ASSUMPTION` labels unchanged, so `/codehelm:audit` can trace them through to `verification.md`. Claude asks you once, with its clarification questions, to approve (they become `ASSUMPTION`) or reject every item it inferred (`PROPOSED`), so the proposed plan contains none. **One task means one proposed reviewable commit:** one concern, a diff you can read in about ten minutes and, apart from setup, a vertical slice that changes observable behavior. Claude plans no speculative "might need" features and no wrappers around framework features, and each new dependency or abstraction gets a decision entry. The plan's **Test commands** (the project's full unit and integration suites, plus its build or lint) are approved with the plan and run after every task, so each task lists only its extra checks. A project without tests gets a setup task first, unless you approve "no integration tests" once for the whole plan.

## 2. Implement one task

```text
/codehelm:implement <plan-name> [task-id]   # for example, /codehelm:implement booking-api S01
```

Without a task ID, Claude picks the next `NOT STARTED` task whose dependencies are done. It reads the approved plan and original sources, checks for drift, confirms every dependency is `DONE` and runs the plan's test commands as a baseline. It stops first only if a source changed since approval, the working tree has uncommitted changes, the task needs work outside its scope or the baseline fails; otherwise it names the task and starts. It writes the tests for the task's acceptance criteria first and confirms they fail, implements exactly one slice, runs every plan test command plus the task's extra checks, fixes failures it caused, checks docs against code, sets the task to `READY FOR HUMAN REVIEW` and **stops for your review**, its one approval stop per task. Reply "accepted" and it sets the task to `DONE` and suggests the commit message; request changes and it applies them, re-runs every check and stops again. The status records the result: `READY FOR HUMAN REVIEW` means every check passed, and `BLOCKED — <reason>` names a cause outside the task, such as a pre-existing failure, missing infrastructure or a needed plan change. The handoff lists the commands and their results; only `/codehelm:audit` writes `verification.md`. It must not stage, commit, push or continue to S02.

Accept before you commit, so the `DONE` status lands in the task's own commit. If you end the session without replying, the task stays `READY FOR HUMAN REVIEW`; a later run that depends on it asks once whether you accepted it.

## 3. Audit

```text
/codehelm:audit <plan-name> <task-id>   # scoped review before the proposed commit
/codehelm:audit <plan-name>             # full review at milestone / delivery
```

Claude checks source requirements ↔ plan ↔ decisions ↔ code ↔ executed tests, then writes grounded evidence to `verification.md`, creating it on the first audit of a plan. It is the only skill that writes this file and the only one that sets `VERIFIED`, and it never treats a claim that tests passed as proof. Each run also appends a row to the file's audit log (date, revision, scope, summary, open findings) and never rewrites earlier rows, so repeated audits keep their history. The audit does **not** change implementation code or create Git commits. Running an audit in a separate fresh Claude session provides more independent review.

## Core rules

- Specify (only if needed), then plan and question first; **no modifications before approval**.
- No automatic branch creation, staging, commits or pushes.
- `/codehelm:implement` and `/codehelm:audit` need a Git repository. Without one, they stop and ask you to run `git init` or to let them run it.
- A missing plan or specification is never guessed. Each skill asks for it and stops. `/codehelm:audit` lists the plans under `docs/plans/`, `/codehelm:plan-tasks` lists the specifications under `docs/specifications/`, and `/codehelm:implement` lists the plans. Given only a plan, `/codehelm:implement` takes its next ready task.
- One approved implementation task per invocation, started only when its dependencies are `DONE`.
- The plan's test commands (full unit **and** integration suites, plus build or lint) run for each proposed commit. If a suite is missing, the plan sets it up first, unless you approved "no integration tests" for the whole plan. A failing baseline stops the task before any edit; afterwards, a missing command or a failure the task did not cause blocks it.
- Document meaningful decisions, keep task-to-requirement traceability and audit code↔docs.
- Keep plans proportionate: a small change gets a lite plan, and small chores shouldn't require invented architectural alternatives.
- If sources or requirements change, pause and request re-approval before proceeding.

## Note on task status

Each task in `plan.md` carries one status: `NOT STARTED`, `BLOCKED — <reason>`, `READY FOR HUMAN REVIEW` or `DONE`. `/codehelm:plan-tasks` sets `NOT STARTED`, and `/codehelm:implement` sets the others.

`READY FOR HUMAN REVIEW` is **not the same as committed or independently VERIFIED**. The engineer controls Git commits and decides when a task is `DONE`. Only `DONE` satisfies a dependency. `/codehelm:audit` never changes a status; it reports mismatches between a status and the evidence as findings, and establishes requirement verification with evidence.

### Compact durable handoffs

Before ending a session, ensure important decisions and task progress are in `plan.md`; audit results are in `verification.md`. The skill should also end with a concise human-readable handoff stating current task/status, tests run, blockers and next command. No separate tracking file is required.

## License

CodeHelm is released under the [MIT License](LICENSE).
