# Verification: <Plan name>

Statuses: `VERIFIED` / `FAILED` / `NOT VERIFIED` / `NOT APPLICABLE`.

Recorded by: `implement (provisional)` for evidence that `/implement` reports about its own task, or `audit` for evidence that `/audit` executed or checked independently. Provisional evidence always stays `NOT VERIFIED`; only `/audit` sets `VERIFIED`, and only on rows recorded by `audit`.

## Audit log

`/audit` appends one row per run and never edits or removes earlier rows. The sections below show the current state; this log keeps the history.

| Date | Revision | Scope | Summary | Open findings |
|---|---|---|---|---|
| — | — | — | No audits yet | — |

*Revision: git SHA or `uncommitted working tree`. Scope: a task ID (`S01`) or `full`. Open findings: count by severity after the run, for example `3 (1 high, 2 medium)` or `0`.*

## Requirements evidence matrix

| Requirement | Source | Planned tasks | Code evidence | Executed test evidence | Docs/decision alignment | Recorded by | Status |
|---|---|---|---|---|---|---|---|
| FR-001 | `<source>#<section>` | S01 | — | — | Not yet evaluated | — | NOT VERIFIED |
| AC-001 | `<source>#<section>` | S01 | — | — | Not yet evaluated | — | NOT VERIFIED |

*One row per requirement and per acceptance criterion, using the same `FR/BR/NFR/AC-###` IDs as `plan.md`, updated in place. `/implement` may fill a row only as `implement (provisional)`; `/audit` switches it to `audit` only with evidence from its own executed checks.*

## Executed checks

| Task | Check | Exact command / CI link | Revision / environment | Observed result | Recorded by | Status |
|---|---|---|---|---|---|---|
| — | — | — | — | No checks recorded yet | — | — |

*Append-only: add one row per check per run and never edit or remove earlier rows; replace only the `No checks recorded yet` placeholder row. Task: a task ID (`S01`), or `full` for a full audit. Check: `Unit tests`, `Integration tests`, `Build / static checks` or a named specialized check.*

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
