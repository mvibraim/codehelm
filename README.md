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
      assets/{plan,decisions,verification}.md
    implement/SKILL.md
    audit/SKILL.md
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

**Upgrading from an earlier copy-based install:** remove the copied `specify`, `plan-tasks`, `implement` and `audit` folders from your project's `.claude/skills/` so they don't compete with the plugin's bare names. Plans you already created keep the old command names (for example `/implement`) in their task status lifecycle section; they still work as bare names, or you can update them by hand.

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

Claude extracts only sourced requirements, identifies gaps, asks **business clarification questions** and **stops**; when stakeholders are unavailable you approve clearly documented assumptions. Claude then drafts a specification without architecture or implementation tasks and **stops again for approval**. After approval it saves:

```text
docs/specifications/<spec-slug>.md
```

The saved file is marked `Approved`, because it is written only after your approval. If you already have an approved specification, skip this phase. If you pass `/codehelm:plan-tasks` a specification that is not marked `Approved`, it warns you and asks whether to plan from it anyway.

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

Claude reads sources, asks focused clarification questions, **stops for your answers**, proposes architecture and small tasks, **stops for approval**, then writes:

```text
docs/plans/<plan-name>/plan.md
docs/plans/<plan-name>/decisions.md
docs/plans/<plan-name>/verification.md
```

It suggests a branch; **you** create/switch branches and commit the planning docs when ready. Respect any confidentiality restrictions on the source documents before committing them.

### Example of small, reviewable tasks

For illustration, a plan for an **Inventory Reservation API** might contain tasks like these. This is **not** a fixed task list: `/codehelm:plan-tasks` must derive the actual tasks from the approved requirements and decisions.

| Task | Small, reviewable change | Example verification |
| --- | --- | --- |
| `S01` | Establish the application, persistence setup, and working test infrastructure | Build succeeds; unit and integration suites run |
| `S02` | Expose a read-only product availability endpoint | Availability unit tests and API/database integration tests |

Each task should have acceptance criteria (`AC-###`), links to applicable requirements, and a suggested commit message. The plan keeps the specification's requirement and acceptance criteria IDs and its `EXPLICIT` / `ASSUMPTION` labels unchanged, so `/codehelm:audit` can trace them through to `verification.md`. Items still `PROPOSED` must be approved (they become `ASSUMPTION`) or rejected before the plan is saved. **One task means one proposed reviewable commit**; the complete unit and integration suites still run after every implementation task. A suite that cannot meaningfully apply to a task (for example, a `docs` change, or a project with no integration boundary) may be marked `NOT APPLICABLE — <reason>` in that task, but only with your explicit approval of the plan or of an amendment that `/codehelm:implement` proposes.

## 2. Implement one task

```text
/codehelm:implement <plan-name> <task-id>   # for example, /codehelm:implement booking-api S01
```

Claude reads the approved plan and original sources, checks for drift, confirms every dependency is `DONE`, proposes the file edits and **stops for your approval**. After approval it implements exactly one slice, runs the complete unit and integration suites (except a suite approved as `NOT APPLICABLE` for the task) plus relevant specialized checks, checks docs against code, sets the task to `READY FOR HUMAN REVIEW` and **stops for your review**. It records its check results in `verification.md` marked `Recorded by: implement (provisional)` and leaves them `NOT VERIFIED`, because only an audit can confirm them. It must not stage, commit, push or continue to S02.

When you have reviewed and accepted the task, mark it `DONE` **before** committing, so the status change lands in the task's own commit:

```text
/codehelm:implement <plan-name> <task-id> --done   # for example, /codehelm:implement booking-api S01 --done
```

If you forget, the next `/codehelm:implement` run asks which `READY FOR HUMAN REVIEW` tasks you have accepted and marks them `DONE` before it changes anything else. When those status lines are the only uncommitted changes in `plan.md`, it recommends committing them on their own (for example, `docs: mark S02 as done`) before you approve its proposal, so the next task's commit holds only its own work. If you skip that, the status changes are part of the next task's diff, and the handoff says so.

## 3. Audit

```text
/codehelm:audit <plan-name> <task-id>   # scoped review before the proposed commit
/codehelm:audit <plan-name>             # full review at milestone / delivery
```

Claude checks source requirements ↔ plan ↔ decisions ↔ code ↔ executed tests, then writes grounded evidence to `verification.md`. It treats provisional evidence from `/codehelm:implement` as claims to re-check, and switches a row to `Recorded by: audit` only with checks it executed itself. Only `/codehelm:audit` sets `VERIFIED`. Each run also appends a row to the file's audit log (date, revision, scope, summary, open findings) and never rewrites earlier rows, so repeated audits keep their history. The audit does **not** change implementation code or create Git commits. Running an audit in a separate fresh Claude session provides more independent review.

## Core rules

- Specify (only if needed), then plan and question first; **no modifications before approval**.
- No automatic branch creation, staging, commits or pushes.
- Git is recommended but not required, and never initialized for you. Without it, the skills skip Git commands. `/codehelm:implement` warns in its proposal that changes can't be reviewed or reverted with Git, and lists every file it changed. `/codehelm:audit` works from the files named in the plan and the recorded evidence.
- Missing arguments are never guessed. Each skill asks for them and stops. `/codehelm:audit` lists the plans under `docs/plans/`, `/codehelm:plan-tasks` lists the specifications under `docs/specifications/`, and `/codehelm:implement` lists the plans or the selected plan's tasks with their statuses.
- One approved implementation task per invocation, started only when its dependencies are `DONE`.
- Full unit **and** integration suites for each proposed commit. If not yet available, bootstrap both suites first. A suite may be skipped only when you approved it as `NOT APPLICABLE — <reason>` for that task; a missing or failing suite otherwise blocks the task.
- Document meaningful decisions, keep task-to-requirement traceability and audit code↔docs.
- Keep plans proportionate; small chores shouldn't require invented architectural alternatives.
- If sources or requirements change, pause and request re-approval before proceeding.

## Note on task status

Each task in `plan.md` carries one status, defined in the plan's "Task status lifecycle" section:

| Status | Meaning | Set by |
| --- | --- | --- |
| `NOT STARTED` | Approved; no work begun | `/codehelm:plan-tasks` |
| `BLOCKED — <reason>` | A mandatory check failed or could not run, or a deviation needs approval | `/codehelm:implement` |
| `READY FOR HUMAN REVIEW` | Implemented and verified locally; not yet accepted by you | `/codehelm:implement` |
| `DONE` | Reviewed and accepted by you | `/codehelm:implement … --done`, or `/codehelm:implement` when you confirm it at the start of a later run |

`READY FOR HUMAN REVIEW` is **not the same as committed or independently VERIFIED**. The engineer controls Git commits and decides when a task is `DONE`. Only `DONE` satisfies a dependency. `/codehelm:audit` never changes a status; it reports mismatches between a status and the evidence as findings, and establishes requirement verification with evidence.

### Compact durable handoffs

Before ending a session, ensure important decisions are in `decisions.md`, task progress in `plan.md` and actual checks in `verification.md`. The skill should also end with a concise human-readable handoff stating current task/status, tests run, blockers and next command. No separate tracking file is required.

## License

CodeHelm is released under the [MIT License](LICENSE).
