# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

CodeHelm is a Claude Code plugin made of Markdown: four skills under `plugins/codehelm/skills/` (`specify`, `plan-tasks`, `implement`, `audit`), each with its templates in `assets/`, plus `README.md` and an MIT `LICENSE`. The repository is also the plugin's marketplace (`.claude-plugin/marketplace.json`). There is no application code, build or test tooling: work here means editing skill prompts, templates, manifests and the README.

Only `plugins/codehelm/` is distributed. Skills must not depend on this file or on anything outside their own skill directory.

The logo is the plugin icon `plugins/codehelm/.claude-plugin/icon.png`, which the README shows. A replacement stays a square PNG of 512–2048 px under 2 MB.

## Commands

- `claude plugin validate ./plugins/codehelm` checks the plugin, and `claude plugin validate .` the marketplace. Run both after changing a manifest or the skill layout.
- `claude --plugin-dir ./plugins/codehelm` loads the working copy for one session, to try out skill changes.

## Invariants

The skills form a pipeline, specify → plan-tasks → implement → audit, and each rule below spans several files. When you change one file, keep the others consistent:

- **Shared preamble.** The "CodeHelm ground rules" and "CodeHelm model and effort" paragraphs are identical in all four `SKILL.md` files. Skills mention `CLAUDE.md` only in the ground rules, because Claude Code loads it anyway.
- **Handoff through files.** Each skill reads the previous phase's Markdown in the target project, never chat history. A handoff that ends a skill's work ends with the **Next** block the model paragraph defines. The README's Git flow section explains the branch and pull-request order once; skills don't.
- **Stops.** Bold **STOP** marks only where a skill waits for the engineer. A skill run without its required argument asks for it and changes no files. Audit runs in a fork and can't ask mid-run, so it ends the run with its question.
- **Shared vocabulary.** Requirement IDs (`FR-`, `BR-`, `NFR-`, `AC-###`), `A-###`, `Q-###`, task IDs (`S01`), `ADR-NNNN`, the provenance labels `EXPLICIT` / `ASSUMPTION` / `PROPOSED`, the task statuses, the audit statuses and the `Task: <plan-name>/<task-id>` commit line appear in skills, templates and the README. Rename them everywhere at once.
- **No `PROPOSED` in saved files.** Specs and plans are saved with only `EXPLICIT` and `ASSUMPTION` items.
- **One writer per file.** specify writes specs. plan-tasks writes plans and ADRs, sets `NOT STARTED`, and changes an accepted ADR only to mark it superseded. implement sets every other task status. Only audit writes `verification.md` and the audit statuses, and it never changes task statuses.
- **Templates hold shape, skills hold rules.** Skills refer to their templates as `${CLAUDE_SKILL_DIR}/assets/...`, never to a path in the target repository.
- **Git and drift.** implement and audit require a Git repository. plan-tasks records the commit at approval in the plan's `Approved` line, and drift is `git diff <sha> -- <source>`.
- **Frontmatter.** Every skill sets `disable-model-invocation: true`. audit adds `context: fork`, `background: false`, `model: opus` and `effort: high`.
- **Command names.** Skills, templates and the README always write commands as `/codehelm:<skill>`.

## Editing conventions

- When you change skill behavior, update the README too (quick start, skills table, rules and Git flow), and the reverse.
- Users stay on the `version` in `plugins/codehelm/.claude-plugin/plugin.json` until it changes. Bump it (semver) when a release should reach users, and keep the plugin description identical in `plugin.json` and `marketplace.json`.
- Before a release, check that `cat $(git ls-files 'plugins/*.md') | wc -w` prints at most 4,000.
- Commit messages use Conventional Commits (`feat:`, `docs:`, ...), as in the existing history.
