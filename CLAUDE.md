# CodeHelm — Engineering Rules for Claude Code

CodeHelm is a **human-controlled** Claude Code workflow with four explicitly invoked skills:

- `/specify`: clarify an incomplete business problem and create an approved, source-traceable specification (optional).
- `/plan-tasks`: clarify and plan an engineering change.
- `/implement`: implement **one** approved, independently reviewable task.
- `/audit`: verify agreement among source requirements, decisions, code, tests and documentation.

These apply to features, fixes, refactors, build/CI work, Docker, observability, performance, tests and docs. **No AI-agent-specific development rules are assumed.**

## Human control and safety

- Never commit, stage, create/switch branches, push, merge, delete files or rewrite Git history without explicit permission for that action. Suggest branch names and commit messages instead.
- Inspect `git status --short` before changes. Do not overwrite or mix in pre-existing uncommitted user changes. Stop and ask if they overlap the task.
- Specification drafting has clarification and explicit draft-approval gates **before writing specification files**.
- Planning has two approval gates: first resolve material questions; then obtain explicit approval of the design and task breakdown **before writing plan documents**.
- Implementation has an approval gate **before editing** and a review gate **after exactly one task**. Never start the next task automatically.
- Do not use destructive commands or modify unrelated files. Flag unexpected changes and blockers.
- Follow the repository's conventions before introducing new ones. Prefer simple designs over unnecessary services, abstractions or dependencies.

## Specifications and decisions

- Specifications are optional. Use `/specify` when the only input is an incomplete business brief; `/plan-tasks` may work directly from a defined change request. Preserve the original assignment as a source rather than overwriting it.
- If an approved project specification exists, it is the source of truth for business behavior; always retain traceability to original instructions. Reconcile conflicts explicitly instead of silently rewriting requirements.
- Preserve existing requirement IDs. Assign stable local IDs when missing, citing the source path and heading (and page when available).
- Keep statements labeled as **explicit requirement**, **approved assumption**, **unapproved proposal** or **engineering choice**. Do not invent missing business rules.
- For each change, maintain `docs/plans/<plan-name>/plan.md`, `decisions.md` and `verification.md` using `docs/templates/`.
- Record important trade-offs and decisions, including rejected alternatives and consequences. Do not fabricate architectural decisions for trivial changes.
- If specifications or approved decisions changed, pause implementation and revise the plan only after approval.

## Reviewable implementation

- Tasks should be as small as practical and leave the project in a working, testable state. Each task must have acceptance criteria, requirement links where relevant and a suggested Conventional Commit message.
- Use the project's documented verification commands. The baseline for **every proposed code commit**, including infra/config changes, is the complete unit and integration test suites, plus appropriate specialized checks.
- If the suites don't exist yet, create minimal working harnesses in the first bootstrap step. Never claim an unavailable or unexecuted suite passed.
- A task is not complete when verification is blocked or failing. Report exact commands and results; never silently skip checks.
- Review changes for correctness, scope, edge cases, security and documentation drift before presenting them for human review.
- Do not automatically create commits or push. The engineer reviews the diff, stages and commits manually.

## Context and session hygiene

- **The repository is the durable memory:** `plan.md`, `decisions.md`, `verification.md` original specifications, code/tests and Git history. Do not rely on a previous chat for an approved requirement or decision.
- Work with focused sessions. Prefer a fresh context for the next independently approved task and a separate session for skeptical audits. Never discard unresolved information before saving it in the appropriate artifact.
- Read only the files, requirements and test outputs relevant to the current task. Avoid unnecessary full-repository scans, huge logs or unrelated tool output.
- Give a short handoff at the end of each specification, plan, implementation and audit: work ID, state, decisions, executed checks, blockers, next human action. Do not create a new handoff file unless necessary.
- Do not hardcode Claude model selection into skills. The engineer can choose a reasoning-heavy model for complex design/audits and a lower-cost coding model for routine implementation; avoid frequent switches within a task.
- Use `/compact` if a long, unfinished investigation needs continuity. Use `/clear` **only after durable work has been saved** and the next task is independent. Inspect `/context` and usage when resource consumption matters.

## Git naming suggestions

Respect existing repository conventions first. Otherwise suggest:
- `feat/<slug>` for features; `fix/<slug>` for defects
- `build/<slug>` for build tooling/Docker; `ci/<slug>` for pipelines
- `chore/<slug>` for maintenance/observability setup
- `refactor/<slug>`, `perf/<slug>`, `test/<slug>`, `docs/<slug>` as appropriate

A branch can include multiple coherent commit types; branch prefixes are suggestions, not hard rules.

## Verification

After implementation, discover and run the project's actual full unit and integration test suites, along with relevant specialized checks, using its native tooling. Do not assume any particular language or wrapper script. Keep CI aligned with those project-specific commands. A local linter cannot prove a hosted CI workflow ran successfully.
