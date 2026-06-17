---
name: sweep
description: Orchestrate the unattended ingest pipeline end to end — detect new raw sources and run compile on each. Use this as the single entry point for automated/batch ingest, typically driven by /loop. It is the orchestrator (team lead) that spawns every phase subagent; no phase agent spawns another.
---

# Ingest Sweep (Automated Pipeline Orchestrator)

One-command orchestration of the automated ingest path. This skill runs on the **main thread** so it can spawn subagents (subagents cannot). It is the **team lead**: it drives `raw-watcher` to discover sources, then runs the full `compile` workflow per source — no phase agent spawns another.

## When to Use

- Unattended / batch ingest of whatever new sources have appeared under `raw/`
- Driven by `/loop` on an interval (see `CLAUDE.md` for the loop command)
- Any time you want "scan raw, compile everything new, and validate" in one step

For ad-hoc, user-driven ingest where you want to choose files interactively, use the `compile` skill instead.

## Input

- No file argument needed — the sweep discovers new sources itself via `raw-watcher`.
- **Non-interactive**: it ingests all detected new sources without asking which ones.
- Default: allow coverage-review self-heal unless the invoker explicitly says review-only.

## Workflow

### Step 1: Detect (raw-watcher)

- Spawn the `raw-watcher` subagent (Agent tool, `subagent_type: raw-watcher`).
- It registers untracked files into `raw/raw-index.md` as `pending` and returns a Verdict with the handoff file list (newly registered + already pending).

### Step 2: Gate on new sources

- If the Verdict is `New sources: no` → report "nothing to ingest" and stop. This is the common, cheap tick.
- If `New sources: yes (N)` → take the N-file list and continue.

### Step 3: Compile each source (finish the whole batch first)

For each source in the list, in source order, run the full compile workflow (**compile Steps 1–8**), non-interactively:

- Skip compile's Step 0 (auto-discover) — `raw-watcher` already discovered and registered the files.
- All other compile steps execute as defined in `.claude/skills/compile/SKILL.md`: extract → analyst team → write wiki pages → build handoff → coverage review.
- Complete all Steps 1–8 for every source before moving on (batch rule: finish ingest for all sources, then coverage review runs per source in source order).

### Step 4: Summary

- Report per source: ingested (source page) + coverage verdict (`Ready: yes/no`).
- List stale scratch files (`.claude/scratch/<source>.txt` and `.claude/scratch/findings/<source>-*.md`) for sources that passed review, so the user can clean them manually.
- List anything skipped (extraction failure, unreadable source, etc.).

## Rules

- **Orchestrator-only spawning.** This skill spawns every subagent; no subagent spawns another. All fan-out and gating happen here, not inside a phase agent.
- **Non-interactive.** Never pause to ask which files — ingest all detected new sources. (The interactive path is the `compile` skill.)
- **Models are fixed in each agent's frontmatter** — `raw-watcher` on Sonnet; the three analysts on Opus 4.8. Do not override per call.
- **Scratch lifecycle is governed by the compile skill rules.** Never move or delete scratch files; surface stale ones in the Step 4 summary.
- This path **writes to the wiki autonomously**; that is intended for the automated / loop use case.
- Treat the sweep as **incomplete** until each source's coverage review finishes.
