# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

CodeHelm is a distributable Claude Code workflow made only of Markdown: four skills under `.claude/skills/`, plus a `README.md`. There is no application code, build, lint or test tooling, so there are no commands to run. Work here means editing skill prompts, their templates and the README.

Users install CodeHelm by copying `.claude/skills/` into their own project (see `README.md`). This `CLAUDE.md` is **not** distributed: skills must not depend on it or on any other file outside their own skill directory.

## Architecture: a four-phase pipeline with human gates

The skills form a pipeline. Each phase hands off to the next only through Markdown files in the *target* project, never through chat history:

| Skill | Reads | Writes (only after explicit approval) |
| --- | --- | --- |
| `/specify` (optional) | a business brief | `docs/specifications/<slug>.md` |
| `/plan-tasks` | a spec or a plain request | `docs/plans/<plan-name>/{plan,decisions,verification}.md` |
| `/implement <plan> <task>` | the plan files | code + tests for exactly one task; updates task status and provisional evidence |
| `/audit <plan> [task]` | plan files, code, tests | only `verification.md` |

Invariants that run through all four skills. Keep them consistent when editing any one of them:

- **Identical portability preamble.** Every `SKILL.md` opens with the same "CodeHelm portability and control" paragraph: respect an existing project `CLAUDE.md` but never create or modify one, never touch Git state (stage/commit/push/branch) without approval, and never initialize Git. When you change this paragraph, change it in all four files.
- **Mandatory STOP points.** Each skill works in phases (A, B, C, ...) with explicit stops for clarification and approval before writing files. `/implement` also stops for human review after verification.
- **Shared vocabulary.** Requirement IDs (`FR-###`, `BR-###`, `NFR-###`, `AC-###`), task IDs (`S01`, `S02`, ...), the provenance labels `EXPLICIT` / `ASSUMPTION` / `PROPOSED`, the task state `READY FOR HUMAN REVIEW`, and the audit statuses `VERIFIED` / `FAILED` / `NOT VERIFIED` / `NOT APPLICABLE`. These terms are defined in several skills and in the `assets/` templates, so a rename must be made everywhere.
- **Bundled templates.** Skills refer to their templates as `${CLAUDE_SKILL_DIR}/assets/...` so that they work after being copied into any project. Never point at a path in the target repo such as `docs/templates/`.
- **Full test suites per task.** `/implement` must run the target project's complete unit and integration suites, discovered from that project's own configuration. CodeHelm ships no wrapper scripts and stays language-agnostic.
- All skills set `disable-model-invocation: true`, so they run only when the user invokes them.

## Editing conventions

- When you change skill behavior, update `README.md` too (skill summaries, the structure tree, the model/session table and the core rules), and the reverse.
- Commit messages use Conventional Commits (`feat:`, `docs:`, ...), as in the existing history.
