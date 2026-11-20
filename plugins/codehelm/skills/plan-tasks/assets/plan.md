# Plan: <plan name>

**Type:** `feat | fix | build | ci | chore | refactor | perf | test | docs`  
**Suggested branch:** `<type>/<slug>`  
**Plan status:** `APPROVED`  
**Plan version:** `1.0`  
**Approval:** `<date / reviewer / reference>`

## Specification sources and versions

| Source | Version / commit / SHA-256 | Sections in scope |
|---|---|---|
| `docs/specifications/<spec-slug>.md` | `<revision>` | `<headings>` |

*If the change has no formal specification, record the originating request and its date as the source. Requirements it states directly are `EXPLICIT`; inferred requirements approved by the engineer are `ASSUMPTION`. If a label differs from the specification's (an item `PROPOSED` there and approved here), note it in this table.*

## Goal and scope

**Goal:** ...

**In scope:** ...

**Out of scope:** ...

## Test commands

- **Unit:** `<command>`
- **Integration:** `<command>`
- **Build / lint:** `<command>`

*Every task runs all of these, plus its own extra checks. Write `Integration: none — <reason> (approved <date>)` only when the engineer approved a plan without integration tests.*

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

`NOT STARTED` → `READY FOR HUMAN REVIEW` → `DONE`. A task whose work had to stop is `BLOCKED — <reason>` until `/codehelm:implement` resumes it.

| Status | Meaning | Set by |
|---|---|---|
| `NOT STARTED` | Approved; no work begun | `/codehelm:plan-tasks` when it writes the plan |
| `BLOCKED — <reason>` | Work stopped until the stated reason is resolved | `/codehelm:implement` when a mandatory check fails or cannot run, or a deviation needs approval |
| `READY FOR HUMAN REVIEW` | Implemented and verified locally; not yet accepted by the engineer and not independently audited | `/codehelm:implement` at the end of Phase C, when every mandatory check passed |
| `DONE` | Reviewed and accepted by the engineer | `/codehelm:implement` when the engineer accepts the task at its review stop, or confirms it at the start of a later run that depends on it |

Only `DONE` satisfies a dependency. Accept the task at the review stop before committing it, so the status change lands in the task's own commit. `/codehelm:audit` reads these statuses and reports mismatches with the evidence, but never changes them.

## Ordered implementation tasks

### S01 — <task title>

- **Status:** `NOT STARTED`
- **Requirement IDs:** `FR-001`
- **Dependencies:** none
- **Scope / files:** ...
- **Acceptance criteria:** `AC-001`
- **Extra checks:** for example, a container build and smoke test
- **Docs updated:** ...
- **Proposed commit:** `feat: ...`

### S02 — <task title>

- **Status:** `NOT STARTED`
- **Requirement IDs:** ...
- **Dependencies:** S01
- **Scope / files:** ...
- **Acceptance criteria:** ...
- **Extra checks:** none
- **Docs updated:** ...
- **Proposed commit:** ...

## Requirements-to-task traceability

| Requirement | Planned task(s) | Coverage |
|---|---|---|
| FR-001 | S01 | Planned |
| AC-001 | S01 | Planned |

## Open risks and agreed limitations

- ...
