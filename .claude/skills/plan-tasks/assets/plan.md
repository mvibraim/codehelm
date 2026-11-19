# Plan: <plan name>

**Type:** `feat | fix | build | ci | chore | refactor | perf | test | docs`  
**Suggested branch:** `<type>/<slug>`  
**Plan status:** `APPROVED`  
**Plan version:** `1.0`  
**Approval:** `<date / reviewer / reference>`

## Specification sources and versions

| Source | Version / commit / SHA-256 | Sections in scope |
|---|---|---|
| `docs/specifications/<file>.md` | `<revision>` | `<headings>` |

*If the change has no formal specification, record the originating request and its date as the source. Requirements it states directly are `EXPLICIT`; inferred requirements approved by the engineer are `ASSUMPTION`. If a label differs from the specification's (an item `PROPOSED` there and approved here), note it in this table.*

## Goal and scope

**Goal:** ...

**In scope:** ...

**Out of scope:** ...

## Requirements and acceptance criteria

| ID | Requirement | Source reference | Basis | Acceptance criteria |
|---|---|---|---|---|
| FR-001 | ... | `<path>#<heading>` | EXPLICIT | AC-001 |
| BR-001 | ... | `<approved assumption, date>` | ASSUMPTION | AC-002 |

| ID | Covers | Scenario (Given / When / Then) | Source reference | Basis |
|---|---|---|---|---|
| AC-001 | FR-001 | Given ... When ... Then ... | `<path>#<heading>` | EXPLICIT |
| AC-002 | BR-001 | Given ... When ... Then ... | `<approved assumption, date>` | ASSUMPTION |

*Keep IDs and labels from the specification unchanged. Basis: `EXPLICIT` (stated in a named source) or `ASSUMPTION` (approved by the engineer). `PROPOSED` items are resolved before approval, so an approved plan contains none.*

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
- **Acceptance criteria:** `AC-001`
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
- **Integration tests:** `NOT APPLICABLE — <reason> (approved <date>)`
- **Other verification:** ...
- **Docs updated:** ...
- **Proposed commit:** ...

*Every task runs the project's full unit and integration suites. A suite may be `NOT APPLICABLE — <reason> (approved <date>)` only with the engineer's explicit approval, given when the plan is approved or later as a plan amendment proposed by `/implement`. `/implement` skips only an approved `NOT APPLICABLE` suite, and skipping it never makes the task `BLOCKED`. A missing or failing suite that is not approved as `NOT APPLICABLE` still blocks the task.*

## Requirements-to-task traceability

| Requirement | Planned task(s) | Coverage |
|---|---|---|
| FR-001 | S01 | Planned |
| AC-001 | S01 | Planned |

## Open risks and agreed limitations

- ...
