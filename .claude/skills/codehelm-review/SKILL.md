---
name: codehelm-review
description: Review the CodeHelm plugin and write a project review and simplification plan. Review only, edits no repository files.
disable-model-invocation: true
---

Review the CodeHelm plugin in this repository and write a project review and simplification plan.

## Context
CodeHelm is a Claude Code plugin with four skills that form a human-gated workflow:
/codehelm:specify → /codehelm:plan-tasks → /codehelm:implement (one task at a time) → /codehelm:audit.
It writes Markdown files into the target repo (specification.md, plan.md, decisions.md, verification.md) from templates bundled via ${CLAUDE_SKILL_DIR}. It has grown and I think it has become too heavy.

## What I want
Tell me honestly what to keep, what to cut and what to fix, so the workflow gets simpler without losing its safeguards. Don't add features unless they close a real gap.

## Read everything first
- All four SKILL.md files, every template, README.md, CLAUDE.md, plugin.json, marketplace.json and the icon.
- Measure the word count of each skill, template and the README, and count repeated rules (e.g. Git prohibitions, "read CLAUDE.md" steps, with/without-Git branches).
- Run `claude plugin validate` on the plugin and the marketplace.

## Research to compare against (cite sources with the date read)
- Anthropic's "Best practices for Claude Code" and "Skills" docs (code.claude.com).

## Structure of the doc
1. **Verdict** – one paragraph, roughly how much text can go, and the top 5 changes in order.
2. **What to keep** – the parts that are the product and must be protected.
3. **What to cut or merge** – a numbered table: item | what it costs today | replace with. Sort by how much text and user effort each removes.
4. **Skill by skill** – the shared preamble (propose a shorter version in a code block), then specify, plan-tasks, implement, audit, with word counts. Call out behavior bugs separately from length problems.
5. **Templates** – templates should hold shape, skills hold rules. Propose slimmed skeletons for plan.md and verification.md in code blocks.
6. **README, CLAUDE.md and manifests** – propose a README outline around a short quick start (with an example session), and check icon size and versioning (semver impact of removed commands).
7. **Agentic coding best practices** – table: practice | CodeHelm today (meets / partly / missing) | action. Gaps first.
8. **Action plan** – checklists grouped into Conventional Commits (cuts, behavior fixes, docs/release), then a plan to validate on one small real project, with a few numbers that show whether it got simpler.

## Style
Plain, direct language. Concrete numbers over adjectives. Keep quotes from sources short. Don't edit any repository files. This is a review only.
