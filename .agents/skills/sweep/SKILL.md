---
name: sweep
description: Orchestrate the unattended ingest pipeline end to end — detect new raw sources and run compile on each. Use this as the single entry point for automated/batch ingest. It is the orchestrator (team lead) that spawns every phase subagent; no phase agent spawns another.
---

# Ingest Sweep (Codex mirror)

This is a **thin mirror**. The canonical, authoritative definition lives at `.claude/skills/sweep/SKILL.md` — read that file first and follow it in full: raw-watcher detection, the per-source compile pipeline (analysts → compile-runner → blind coverage review), batch ordering, and the summary format. Do not fork workflow content here; update the canonical file instead (see `rules/repository-structure.md` → Single Source of Truth).

Codex-specific notes:

- Spawn the underscore-named Codex sub-agents from `.codex/agents/` (`raw_watcher`, the three analysts, `compile_runner`, `coverage_reviewer`, `blind_answerer`) wherever the canonical skill names the hyphenated Claude agents.
