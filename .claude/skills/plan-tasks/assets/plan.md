# Plan: <plan name>

**Type:** `feat | fix | build | ci | chore | refactor | perf | test | docs`  
**Suggested branch:** `<type>/<slug>`  
**Status:** `APPROVED | PROPOSED`  
**Approval:** `<date / reviewer / reference>`

## Specification sources and versions

| Source | Version / commit / SHA-256 | Sections in scope |
|---|---|---|
| `docs/specifications/<file>.md` | `<revision>` | `<headings>` |

*If the change has no formal specification, record the originating task/request and label requirements as proposed/approved assumptions.*

## Goal and scope

**Goal:** ...

**In scope:** ...

**Out of scope:** ...

## Requirements and acceptance criteria

| ID | Explicit requirement or approved assumption | Source reference | Acceptance criteria |
|---|---|---|---|
| FR-001 | ... | `<path>#<heading>` | ... |

## Architecture and implementation approach

Brief component/data flow description. Reference decision IDs from `decisions.md`; don't duplicate detailed rationale.

## Task status lifecycle

`NOT STARTED` → `READY FOR HUMAN REVIEW` → `DONE`. A task whose work had to stop is `BLOCKED — <reason>` until `/implement` resumes it.

| Status | Meaning | Set by |
|---|---|---|
| `NOT STARTED` | Approved; no work begun | `/plan-tasks` when it writes the plan |
| `BLOCKED — <reason>` | Work stopped until the stated reason is resolved | `/implement` when a mandatory check fails or cannot run, or a deviation needs approval |
| `READY FOR HUMAN REVIEW` | Implemented and verified locally; not yet accepted by the engineer and not independently audited | `/implement` at the end of Phase C, when every mandatory check passed |
| `DONE` | Reviewed and accepted by the engineer | `/implement <plan-name> <task-id> --done`, or `/implement` when the engineer confirms it at the start of a later run |

Only `DONE` satisfies a dependency. Run `--done` before committing the task, so the status change lands in the task's own commit. `/audit` reads these statuses and reports mismatches with the evidence, but never changes them.

## Ordered implementation tasks

### S01 — <task title>

- **Status:** `NOT STARTED`
- **Requirement IDs:** `FR-001`
- **Dependencies:** none
- **Scope / files:** ...
- **Acceptance criteria:** ...
- **Unit tests:** full suite plus new/updated cases ...
- **Integration tests:** full suite plus new/updated cases ...
- **Other verification:** build / Docker smoke / workflow validation as relevant
- **Docs updated:** ...
- **Proposed commit:** `feat: ...`

### S02 — <task title>

- **Status:** `NOT STARTED`
- **Requirement IDs:** ...
- **Dependencies:** S01
- **Scope / files:** ...
- **Acceptance criteria:** ...
- **Unit tests:** ...
- **Integration tests:** ...
- **Other verification:** ...
- **Docs updated:** ...
- **Proposed commit:** ...

## Requirements-to-task traceability

| Requirement | Planned task(s) | Coverage |
|---|---|---|
| FR-001 | S01 | Planned |

## Open risks and agreed limitations

- ...
