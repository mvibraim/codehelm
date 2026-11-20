# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

CodeHelm is a Claude Code plugin made of Markdown: four skills under `plugins/codehelm/skills/`, plus a `README.md` and an MIT `LICENSE`. This repository is both the plugin marketplace (`.claude-plugin/marketplace.json`, marketplace `codehelm`) and the home of the plugin (`plugins/codehelm/.claude-plugin/plugin.json`, plugin `codehelm`). The logo is stored once, as the plugin icon `plugins/codehelm/.claude-plugin/icon.png`, and the README shows that file. Plugin paths can't point outside the plugin folder, so the logo lives there rather than elsewhere in the repo. A replacement must stay a square PNG of 512–2048 px under 2 MB. There is no application code, build or test tooling. Work here means editing skill prompts, their templates, the manifests and the README.

Only `plugins/codehelm/` is distributed. This `CLAUDE.md` is **not**: skills must not depend on it or on any other file outside their own skill directory.

Commands:

- `claude plugin validate ./plugins/codehelm` checks the plugin, and `claude plugin validate .` checks the marketplace. Run both after changing a manifest or the skill layout.
- `claude --plugin-dir ./plugins/codehelm` loads the working copy for one session, to try out skill changes.

## Architecture: a four-phase pipeline with human gates

The skills form a pipeline. Each phase hands off to the next only through Markdown files in the *target* project, never through chat history:

| Skill | Reads | Writes |
| --- | --- | --- |
| `/codehelm:specify` (optional) | a business brief | `docs/specifications/<spec-slug>.md`, after approval |
| `/codehelm:plan-tasks` | a spec or a plain request | `docs/plans/<plan-name>/plan.md`, after approval: a full plan, or a lite plan for one or two tasks with no real design choice |
| `/codehelm:implement <plan> [task]` | `plan.md` and the linked spec sections | tests first, then code, for exactly one task, plus that task's status line. The plan's approval covers the task. Sets `DONE` only when the engineer accepts the task at its review stop, or confirms a `READY FOR HUMAN REVIEW` dependency at the start of a run |
| `/codehelm:audit <plan> [task]` | `plan.md`, `verification.md`, code, tests | only `verification.md`, without an approval stop, created from `audit/assets/verification.md` on the first run |

Invariants that run through all four skills. Keep them consistent when editing any one of them:

- **Identical ground rules.** Every `SKILL.md` opens with the same "CodeHelm ground rules" paragraph: follow the project's `CLAUDE.md` and conventions but never create or edit `CLAUDE.md`, never stage, commit, push, switch branches, rewrite history or delete work unless the engineer asks, keep the engineer's uncommitted changes, change an approved spec or plan only with approval, and ask when a project instruction conflicts with these rules. Skills mention `CLAUDE.md` only in this paragraph, because Claude Code already loads it. When you change this paragraph, change it in all four files.
- **Mandatory STOP points.** Each skill works in phases (A, B, C, ...) with explicit stops, and a skill run without its required argument asks for it and changes no files.
  - `/codehelm:specify` and `/codehelm:plan-tasks` stop for clarification and again for approval before writing files. Each clarification round asks at most five numbered questions, each with options and a recommended answer.
  - `/codehelm:implement` starts directly, because the approved plan covers its task. Given only a plan name, it picks the next `NOT STARTED` task whose dependencies are `DONE`, after asking about any that are still `READY FOR HUMAN REVIEW`. It stops first only on drift since approval, a dirty tree, work outside the task's scope, a plan conflict or a failing baseline. It always stops for human review after verification, and after each requested change re-runs the checks and stops again until the engineer accepts.
  - `/codehelm:audit` has no approval stop. It writes only `verification.md`, and ends the run with a question wherever it must ask.
- **Git and drift.** `/codehelm:implement` and `/codehelm:audit` require a Git repository and stop if there is none. `/codehelm:plan-tasks` records the commit at approval in the plan's `Approved` line, and implement and audit detect source drift with `git diff <sha> -- <source>`: implement stops, and audit reports it as a finding.
- **Shared vocabulary.** Requirement IDs (`FR-###`, `BR-###`, `NFR-###`, `AC-###`), task IDs (`S01`, `S02`, ...), decision IDs (`ADR-###`, in the Decisions section of `plan-tasks/assets/plan.md`), assumption IDs (`A-###`) and open-question IDs (`Q-###`, both in `specify/assets/specification.md`), the spec filename placeholder `<spec-slug>` (`docs/specifications/<spec-slug>.md`), the provenance labels `EXPLICIT` / `ASSUMPTION` / `PROPOSED`, the task statuses `NOT STARTED` / `BLOCKED — <reason>` / `READY FOR HUMAN REVIEW` / `DONE` (defined in `/codehelm:implement`, which sets all of them except `NOT STARTED`, set by `/codehelm:plan-tasks`; `/codehelm:audit` restates them to check them and never changes them), and the audit statuses `VERIFIED` / `FAILED` / `NOT VERIFIED` / `NOT APPLICABLE` (only `/codehelm:audit` writes `verification.md` and sets them). These terms are defined in several skills and in the skills' bundled `assets/` templates, so a rename must be made everywhere.
- **No `PROPOSED` in saved files.** `PROPOSED` marks an unapproved item only while a skill runs. Saved specs and plans contain only `EXPLICIT` and `ASSUMPTION` items: `/codehelm:specify` turns an approved item into an `ASSUMPTION` with an `A-###` entry and an unapproved one into a `Q-###` open question, and `/codehelm:plan-tasks` asks once, in Phase B, making each item an `ASSUMPTION` or moving it out of scope.
- **Two plan shapes.** A lite plan has only the `Approved` and `Sources` lines, goal and scope, test commands and tasks, with each task's acceptance criteria in its **Covers** line. `/codehelm:implement` and `/codehelm:audit` must work with both lite and full plans.
- **Bundled templates.** Skills refer to their templates as `${CLAUDE_SKILL_DIR}/assets/...` so that they work wherever the plugin is installed. Never point at a path in the target repo such as `docs/templates/`.
- **Plan-level test commands.** `/codehelm:plan-tasks` discovers the target project's complete unit and integration suites and its build or lint command from that project's own configuration and records them in the plan's **Test commands** section, so approving the plan approves them. A project without tests gets a setup task first, unless the engineer approves "no integration tests" once for the whole plan. Each task lists only its **Extra checks**. `/codehelm:implement` runs the test commands as a baseline before editing and stops if one fails, then runs every test command plus the task's extra checks after implementing. It fixes failures it caused; a missing command or a failure from outside the task sets `BLOCKED`. `/codehelm:audit` runs the same commands when it needs evidence, and reports a suite the changes need that the plan lacks. CodeHelm ships no wrapper scripts and stays language-agnostic.
- All skills set `disable-model-invocation: true`, so they run only when the user invokes them. `/codehelm:audit` also sets `context: fork` with `background: false`: it runs in a subagent without the conversation history and can't ask questions mid-run, so wherever it must ask, it ends the run with the question.
- **Namespaced command names.** Skills, templates and the README always write commands in the plugin form `/codehelm:<skill>`, including next-command hints.

## Editing conventions

- When you change skill behavior, update `README.md` too (the quick start, the skill summaries, the task status line, the rules and the tips), and the reverse.
- Users stay on the `version` in `plugins/codehelm/.claude-plugin/plugin.json` until it changes. Bump it (semver) when a release should reach users, and keep the plugin description in `plugin.json` and `marketplace.json` the same.
- Commit messages use Conventional Commits (`feat:`, `docs:`, ...), as in the existing history.
