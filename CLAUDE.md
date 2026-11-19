# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

CodeHelm is a distributable Claude Code workflow made of Markdown: four skills under `.claude/skills/`, plus a `README.md` and an MIT `LICENSE`. The `.github/assets/` folder holds images used only by the README, such as `.github/assets/codehelm-logo.png`. It is not distributed and is separate from the skills' own `assets/` template folders. There is no application code, build, lint or test tooling, so there are no commands to run. Work here means editing skill prompts, their templates and the README.

Users install CodeHelm by copying `.claude/skills/` into their own project (see `README.md`). This `CLAUDE.md` is **not** distributed: skills must not depend on it or on any other file outside their own skill directory.

## Architecture: a four-phase pipeline with human gates

The skills form a pipeline. Each phase hands off to the next only through Markdown files in the *target* project, never through chat history:

| Skill | Reads | Writes (only after explicit approval) |
| --- | --- | --- |
| `/specify` (optional) | a business brief | `docs/specifications/<slug>.md` |
| `/plan-tasks` | a spec or a plain request | `docs/plans/<plan-name>/{plan,decisions,verification}.md` |
| `/implement <plan> <task> [--done]` | the plan files | code + tests for exactly one task; updates task status and provisional evidence. With `--done`, or when the engineer confirms an earlier task at the start of a run, it only sets that task to `DONE` |
| `/audit <plan> [task]` | plan files, code, tests | only `verification.md` |

Invariants that run through all four skills. Keep them consistent when editing any one of them:

- **Identical portability preamble.** Every `SKILL.md` opens with the same "CodeHelm portability and control" paragraph: respect an existing project `CLAUDE.md` but never create or modify one, never touch Git state (stage/commit/push/branch) without approval, and never initialize Git. When you change this paragraph, change it in all four files.
- **Mandatory STOP points.** Each skill works in phases (A, B, C, ...) with explicit stops for clarification and approval before writing files. `/implement` also stops for human review after verification.
- **Shared vocabulary.** Requirement IDs (`FR-###`, `BR-###`, `NFR-###`, `AC-###`), task IDs (`S01`, `S02`, ...), the provenance labels `EXPLICIT` / `ASSUMPTION` / `PROPOSED`, the task statuses `NOT STARTED` / `BLOCKED — <reason>` / `READY FOR HUMAN REVIEW` / `DONE` (defined once, in the "Task status lifecycle" section of `plan-tasks/assets/plan.md`, together with which skill sets each one), the audit statuses `VERIFIED` / `FAILED` / `NOT VERIFIED` / `NOT APPLICABLE`, and the evidence markers `Recorded by: implement (provisional)` / `audit` in `verification.md` (only `/audit` sets `VERIFIED`, and only on rows it recorded). These terms are defined in several skills and in the skills' bundled `assets/` templates, so a rename must be made everywhere.
- **Bundled templates.** Skills refer to their templates as `${CLAUDE_SKILL_DIR}/assets/...` so that they work after being copied into any project. Never point at a path in the target repo such as `docs/templates/`.
- **Full test suites per task.** `/implement` must run the target project's complete unit and integration suites, discovered from that project's own configuration. CodeHelm ships no wrapper scripts and stays language-agnostic.
- All skills set `disable-model-invocation: true`, so they run only when the user invokes them.

## Editing conventions

- When you change skill behavior, update `README.md` too (skill summaries, the structure tree, the model/session table and the core rules), and the reverse.
- Commit messages use Conventional Commits (`feat:`, `docs:`, ...), as in the existing history.
