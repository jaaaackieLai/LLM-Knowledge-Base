---
name: coverage-review
description: Use when the user asks to evaluate whether the LLM Knowledge Base wiki contains enough information to answer source-grounded questions, run a post-compile coverage check, or validate a compile handoff payload. This skill identifies answerability gaps from wiki pages only, self-heals high-confidence omissions, and validates repairs with regression plus holdout questions.
---

# Coverage Review (Codex mirror)

This is a **thin mirror**. The canonical, authoritative definition lives at `.claude/skills/coverage-review/SKILL.md` — read that file first and follow it in full: the three-role blind protocol (reviewer / blind answerer / orchestrator), scratch file contract, grading rubric, thresholds, self-heal policy, and report format. Do not fork workflow content here; update the canonical file instead (see `rules/repository-structure.md` → Single Source of Truth).

Codex-specific notes:

- The reviewer role runs as the `coverage_reviewer` sub-agent and every blind pass as a fresh `blind_answerer` sub-agent (definitions in `.codex/agents/`). The orchestrating context relays scratch-file paths between them and never lets a context that has read the raw source run the answer pass.
