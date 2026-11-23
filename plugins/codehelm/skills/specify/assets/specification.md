# Spec: [Feature name]

Approved: <date> · Source: <path>, or § Source brief

## Problem and desired outcome

**Problem:** [Business problem and consequences]

**Goal:** [Expected outcome]

**Users / interacting systems:** [Relevant actors]

## Scope

**In scope:** [Requested functionality]

**Out of scope:** [Explicitly excluded functionality; distinguish exclusions from assumptions]

## Functional requirements

| ID | Observable behavior | Provenance / source | Basis |
|---|---|---|---|
| FR-001 | The system must ... | `[brief.md § Section]` | EXPLICIT |
| FR-002 | The system must ... | `[User approved assumption A-001]` | ASSUMPTION |

## Business rules and invariants

| ID | Rule or invariant | Provenance / basis |
|---|---|---|
| BR-001 | [What must always hold] | [Source or approved assumption] |

## Nonfunctional requirements

| ID | Quality expectation or constraint | Provenance / basis |
|---|---|---|
| NFR-001 | [Correctness, consistency, security, performance, etc.] | [Source or approved assumption] |

## Acceptance scenarios

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

## Constraints and dependencies

**Mandatory technology / environment:** [Technology the source imposes]

**External dependencies / data:** [Systems and data the feature relies on, including the other feature specs this one depends on, as `docs/specs/<spec-slug>.md`]

**Delivery expectations:** [Delivery restrictions the source states]

## Approved assumptions

| ID | Assumption and why it was needed | Approved by / when | Affected requirements |
|---|---|---|---|
| A-001 | [Assumption that wasn't specified externally] | [Engineer + date] | FR-002 |

## Open questions and limitations

| ID | Question / uncertainty | Impact | Status |
|---|---|---|---|
| Q-001 | [Unresolved business behavior] | [Blocker or nonblocking] | OPEN |

## Source brief

[Only for a brief given as chat text: the brief, quoted unchanged]
