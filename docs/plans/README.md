# Engineering plans

For each approved change, create `docs/plans/<plan-name>/` with:

- `plan.md`: sources, requirements, approved task breakdown, acceptance criteria and test expectations.
- `decisions.md`: accepted design choices and trade-offs; short if no significant decisions.
- `verification.md`: requirement-to-code-and-test evidence and outstanding audit findings.

Start from `docs/templates/`. These files are **outputs of `/plan-tasks` after approval**, not prerequisites for the skill. Keep original business specifications in `docs/specifications/` or at an accessible external path.

Example commands:

```text
/plan-tasks docs/specifications/<spec>.md
/plan-tasks Add a Dockerfile and local container smoke test
/implement <plan-name> <task-id>
/audit <plan-name> <task-id>
/audit <plan-name>
```
