# [Project Name] — Business Specification

**Status:** Draft | **Version:** 0.1 | **Updated:** YYYY-MM-DD  
**Source:** [Original brief or requirement file; original stays unchanged]

> This describes **what** the system must do. Technical design, implementation tasks and Git commits are produced later by `/plan-tasks`.

## 1. Problem and desired outcome

**Problem:** [Business problem and consequences]

**Goal:** [Expected outcome; avoid inventing metrics]

**Users / interacting systems:** [Relevant actors]

## 2. Scope

**In scope:** [Requested capabilities]

**Out of scope:** [Explicitly excluded capabilities; distinguish exclusions from assumptions]

## 3. Functional requirements

| ID | Observable behavior | Provenance / source | Basis |
|---|---|---|---|
| FR-001 | The system must ... | `[brief.md § Section]` | EXPLICIT |
| FR-002 | The system must ... | `[User approved assumption A-001]` | ASSUMPTION |

Basis: `EXPLICIT` (documented source), `ASSUMPTION` (approved by engineer) or `PROPOSED` (awaiting approval; must not be treated as final).

## 4. Business rules and invariants

| ID | Rule or invariant | Provenance / basis |
|---|---|---|
| BR-001 | [What must always hold] | [Source or approved assumption] |

## 5. Nonfunctional requirements

| ID | Quality expectation or constraint | Provenance / basis |
|---|---|---|
| NFR-001 | [Correctness, consistency, security, performance, etc.] | [Source or approved assumption] |

If a numeric target is unknown, say `TBD`, not an invented number.

## 6. Acceptance scenarios

### AC-001 — [Meaningful scenario]

**Covers:** FR-001, BR-001  
**Given** [initial state]  
**When** [action]  
**Then** [observable result]  
**Source:** [Reference or approved assumption]

### AC-002 — [Failure / retry / edge case]

**Covers:** [linked requirements]  
**Given** ...  
**When** ...  
**Then** ...

## 7. Constraints and dependencies

**Mandatory technology / environment:** [Only when required by the source]

**External dependencies / data:** [Only when relevant]

**Delivery expectations:** [Only when required by the source]

## 8. Approved assumptions

| ID | Assumption and why it was needed | Approved by / when | Affected requirements |
|---|---|---|---|
| A-001 | [Assumption that wasn't specified externally] | [Engineer + date] | FR-002 |

If none, write `None`.

## 9. Open questions and limitations

| ID | Question / uncertainty | Impact | Status |
|---|---|---|---|
| Q-001 | [Unresolved business behavior] | [Blocker or nonblocking] | OPEN |

---

**Next step after approval:** `/plan-tasks docs/specifications/<project-slug>.md`.
