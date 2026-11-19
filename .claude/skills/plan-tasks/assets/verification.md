# Verification: <Plan name>

Statuses: `VERIFIED` / `FAILED` / `NOT VERIFIED` / `NOT APPLICABLE`.

## Audit log

`/audit` appends one row per run and never edits or removes earlier rows. The sections below show the current state; this log keeps the history.

| Date | Revision | Scope | Summary | Open findings |
|---|---|---|---|---|
| — | — | — | No audits yet | — |

*Revision: git SHA or `uncommitted working tree`. Scope: a task ID (`S01`) or `full`. Open findings: count by severity after the run, for example `3 (1 high, 2 medium)` or `0`.*

## Requirements evidence matrix

| Requirement | Source | Planned tasks | Code evidence | Executed test evidence | Docs/decision alignment | Status |
|---|---|---|---|---|---|---|
| FR-001 | `<source>#<section>` | S01 | — | — | Not yet evaluated | NOT VERIFIED |

## Executed checks

| Check | Exact command / CI link | Revision / environment | Observed result | Status |
|---|---|---|---|---|
| Unit tests | — | — | — | NOT VERIFIED |
| Integration tests | — | — | — | NOT VERIFIED |
| Build / static checks | — | — | — | NOT VERIFIED |
| Specialized checks | — | — | — | NOT VERIFIED |

## Findings and follow-ups

| Severity | Requirement / task | Finding and evidence | Proposed next action | Status |
|---|---|---|---|---|
| — | — | — | — | — |

## Review conclusion

What is verified, what is blocked and what needs human review? Never infer `VERIFIED` from code existence alone.

## Fresh-session handoff (optional when work is interrupted)

Use this short block only if a task cannot finish and important context is not yet captured in the plan/decisions/evidence above.

- **Current task and state:** ...
- **Last meaningful finding or approved decision:** ...
- **Verification completed / blocked:** ...
- **Next human decision or exact command:** ...
