# Specifications

Place business requirements and original specification documents here. You can also supply local external file paths to `/plan-tasks`.

- Specifications are authoritative for business behavior; planning and ADRs document *engineering* decisions.
- Preserve original wording and IDs where possible.
- If specifications change, rerun `/plan-tasks` to assess and approve impacts before implementing affected tasks.
- For PDF/DOCX specifications, cite the original file plus its heading/page as available; do not rely on a lossy conversion without review.

## Creating a specification from a brief

1. Keep the original assignment unchanged (for example `docs/assignments/<name>.md` or outside the repository if confidential).
2. Run `/specify <path-to-brief>` or `/specify <problem description>`.
3. Answer prioritized clarification questions and explicitly approve assumptions.
4. Review and approve the generated draft. Only then is it saved as `docs/specifications/<spec>.md` using `docs/templates/specification.md`.
5. Run `/plan-tasks docs/specifications/<spec>.md` to design the solution and create implementation tasks.

`/specify` documents **business behavior**, not architecture, commits or implementation steps.
