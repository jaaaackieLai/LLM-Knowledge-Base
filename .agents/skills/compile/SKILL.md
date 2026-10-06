---
name: compile
description: Use when the user asks to compile, ingest, or turn raw LLM Knowledge Base source files into wiki pages, including updating a domain's source summaries, concepts, entities, questions, entrances, log, and raw tracking. By default, after ingest, run a fresh post-compile coverage review unless the user explicitly asks for compile-only.
---

# Compile Raw Source into Wiki (Codex mirror)

This is a **thin mirror**. The canonical, authoritative definition of this skill lives at `.claude/skills/compile/SKILL.md` — read that file first and follow it in full: orchestrator workflow (Steps 0–5), Write Phase Specification (W1–W5), scratch file naming, and rules. Do not fork workflow content here; if something needs to change, change the canonical file (see `rules/repository-structure.md` → Single Source of Truth).

Codex-specific notes:

- Sub-agent definitions live in `.codex/agents/` with underscore names: `raw_watcher`, `theory_context_analyst`, `derivation_checker`, `experiment_synthesizer`, `compile_runner`, `coverage_reviewer`, `blind_answerer`. Wherever the canonical skill says to spawn a hyphenated Claude subagent, spawn the corresponding Codex sub-agent.
- Codex has no `paper-reading` skill and no `paper_reader` sub-agent. When a domain's `GUIDE.md` → Analysts lists `paper-reader`, spawn `experiment_synthesizer` with the findings path `<slug>-experiment.md` in its place, pass that path to `compile_runner`, and tell the user that the substitution ran.
- Everything else — phase order, exact scratch paths, blind-review protocol, thresholds, boundaries, no-deletion policy — is exactly as the canonical file specifies.
