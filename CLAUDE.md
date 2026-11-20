# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

CodeHelm is a Claude Code plugin made of Markdown: four skills under `plugins/codehelm/skills/`, plus a `README.md` and an MIT `LICENSE`. This repository is both the plugin marketplace (`.claude-plugin/marketplace.json`, marketplace `codehelm`) and the home of the plugin (`plugins/codehelm/.claude-plugin/plugin.json`, plugin `codehelm`). The logo is stored once, as the plugin icon `plugins/codehelm/.claude-plugin/icon.png`, and the README shows that file. Plugin paths can't point outside the plugin folder, so the logo lives there rather than elsewhere in the repo. A replacement must stay a square PNG of 512–2048 px under 2 MB. There is no application code, build or test tooling. Work here means editing skill prompts, their templates, the manifests and the README.

Users install CodeHelm with `/plugin install codehelm --marketplace mvibraim/codehelm` (see `README.md`), and run its skills as `/codehelm:<skill>`. Only `plugins/codehelm/` is distributed. This `CLAUDE.md` is **not**: skills must not depend on it or on any other file outside their own skill directory.

Commands:

- `claude plugin validate ./plugins/codehelm` checks the plugin, and `claude plugin validate .` checks the marketplace. Run both after changing a manifest or the skill layout.
- `claude --plugin-dir ./plugins/codehelm` loads the working copy for one session, to try out skill changes.

## Architecture: a four-phase pipeline with human gates

The skills form a pipeline. Each phase hands off to the next only through Markdown files in the *target* project, never through chat history:

| Skill | Reads | Writes (only after explicit approval) |
| --- | --- | --- |
| `/codehelm:specify` (optional) | a business brief | `docs/specifications/<spec-slug>.md` |
| `/codehelm:plan-tasks` | a spec or a plain request | `docs/plans/<plan-name>/{plan,decisions}.md` |
| `/codehelm:implement <plan> <task> [--done]` | the plan files | code + tests for exactly one task; updates task status. With `--done`, or when the engineer confirms an earlier task at the start of a run, it only sets that task to `DONE` |
| `/codehelm:audit <plan> [task]` | plan files, code, tests | only `verification.md`, created from `audit/assets/verification.md` on the first run |

Invariants that run through all four skills. Keep them consistent when editing any one of them:

- **Identical ground rules.** Every `SKILL.md` opens with the same "CodeHelm ground rules" paragraph: follow the project's `CLAUDE.md` and conventions but never create or edit `CLAUDE.md`, never stage, commit, push, switch branches, rewrite history or delete work unless the engineer asks, keep the engineer's uncommitted changes, change an approved spec or plan only with approval, and ask when a project instruction conflicts with these rules. Skills mention `CLAUDE.md` only in this paragraph, because Claude Code already loads it. When you change this paragraph, change it in all four files.
- **Mandatory STOP points.** Each skill works in phases (A, B, C, ...) with explicit stops for clarification and approval before writing files. `/codehelm:implement` also stops for human review after verification.
- **Shared vocabulary.** Requirement IDs (`FR-###`, `BR-###`, `NFR-###`, `AC-###`), task IDs (`S01`, `S02`, ...), decision IDs (`ADR-###`, in `plan-tasks/assets/decisions.md`), assumption IDs (`A-###`) and open-question IDs (`Q-###`, both in `specify/assets/specification.md`), the spec filename placeholder `<spec-slug>` (`docs/specifications/<spec-slug>.md`), the provenance labels `EXPLICIT` / `ASSUMPTION` / `PROPOSED`, the task statuses `NOT STARTED` / `BLOCKED — <reason>` / `READY FOR HUMAN REVIEW` / `DONE` (defined once, in the "Task status lifecycle" section of `plan-tasks/assets/plan.md`, together with which skill sets each one), and the audit statuses `VERIFIED` / `FAILED` / `NOT VERIFIED` / `NOT APPLICABLE` (only `/codehelm:audit` writes `verification.md` and sets them). These terms are defined in several skills and in the skills' bundled `assets/` templates, so a rename must be made everywhere.
- **Bundled templates.** Skills refer to their templates as `${CLAUDE_SKILL_DIR}/assets/...` so that they work wherever the plugin is installed. Never point at a path in the target repo such as `docs/templates/`.
- **Plan-level test commands.** `/codehelm:plan-tasks` discovers the target project's complete unit and integration suites and its build or lint command from that project's own configuration and records them in the plan's **Test commands** section, so approving the plan approves them. A project without tests gets a setup task first, unless the engineer approves "no integration tests" once for the whole plan. Each task lists only its **Extra checks**. `/codehelm:implement` runs every test command plus the task's extra checks after implementing; a missing or failing one sets `BLOCKED`. CodeHelm ships no wrapper scripts and stays language-agnostic.
- All skills set `disable-model-invocation: true`, so they run only when the user invokes them.
- **Namespaced command names.** Skills, templates and the README always write commands in the plugin form `/codehelm:<skill>`, including next-command hints and the task status lifecycle table in `plan-tasks/assets/plan.md`.

## Editing conventions

- When you change skill behavior, update `README.md` too (skill summaries, the structure tree, the model/session table and the core rules), and the reverse.
- Users stay on the `version` in `plugins/codehelm/.claude-plugin/plugin.json` until it changes. Bump it (semver) when a release should reach users, and keep the plugin description in `plugin.json` and `marketplace.json` the same.
- Commit messages use Conventional Commits (`feat:`, `docs:`, ...), as in the existing history.
