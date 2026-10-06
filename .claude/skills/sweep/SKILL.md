---
name: sweep
description: Orchestrate the unattended ingest pipeline end to end — detect new raw sources and run compile on each. Use this as the single entry point for automated/batch ingest, typically driven by /loop. It is the orchestrator (team lead) that spawns every phase subagent; no phase agent spawns another.
---

# Ingest Sweep (Automated Pipeline Orchestrator)

One-command orchestration of the automated ingest path. This skill runs on the **main thread** so it can spawn subagents (subagents cannot). It is the **team lead**: it drives `raw-watcher` to discover sources, then runs the full `compile` workflow per source — no phase agent spawns another. The main thread never reads the raw source or writes wiki pages itself; the analysts, compile-runner, coverage-reviewer, and blind-answerer do the reading and writing.

## When to Use

- Unattended / batch ingest of whatever new sources have appeared under any domain's `<domain>/raw/` (a domain is a root folder that contains `GUIDE.md`)
- Driven by `/loop` on an interval (see `AGENTS.md` for the default cadence)
- Any time you want "scan every domain's raw folder, compile everything new, and validate" in one step

For ad-hoc, user-driven ingest where you want to choose files interactively, use the `compile` skill instead.

## Input

- No file argument needed — the sweep discovers new sources itself via `raw-watcher`.
- **Non-interactive**: it ingests all detected new sources without asking which ones.
- Default: allow coverage-review self-heal unless the invoker explicitly says review-only.

## Workflow

### Step 1: Detect (raw-watcher)

- Spawn the `raw-watcher` subagent (Agent tool, `subagent_type: raw-watcher`).
- It scans every domain (`*/GUIDE.md`), registers untracked files into that domain's `<domain>/raw/raw-index.md` as `pending`, self-checks each table after appending, and returns a Verdict with the handoff file list as `<domain>/raw/<file>` paths (newly registered + already pending).

### Step 2: Gate on new sources

- If the Verdict is `New sources: no` → report "nothing to ingest" and stop. This is the common, cheap tick.
- If `New sources: yes (N)` → take the N-file list and continue.

### Step 3: Compile each source (finish the whole batch first)

For each source in the list, in source order, run the compile skill's orchestrator workflow (**compile Steps 1–5**), non-interactively:

- Skip compile's Step 0 (auto-discover) — `raw-watcher` already discovered and registered the files.
- Per source: verify target and read `<domain>/GUIDE.md` (Step 1) → spawn the domain's analyst team in parallel with exact findings paths, including the paper-reader fallback (Step 2) → spawn compile-runner with the findings paths (Step 3) → collect the handoff payload and relay any Claim–Evidence Conflicts (Step 4).
- Batch rule: finish Steps 1–4 for **every** source first; then run the blind coverage-review protocol (compile Step 5) per source in source order.

### Step 4: Summary

- Report per source, grouped by domain: ingested (source page) + coverage verdict (`Ready: yes/no`) + any claim–evidence conflicts.
- List stale scratch files (`.claude/scratch/findings/<slug>-*.md`, `.claude/scratch/coverage/<slug>-*.md`, and paper-reader code clones under `.claude/scratch/code/<slug>/`) for sources that passed review, so the user can clean them manually.
- List anything skipped (unreadable source, missing `pdftoppm`, etc.) and every paper-reader fallback to experiment-synthesizer.

## Rules

- **Orchestrator-only spawning.** This skill spawns every subagent; no subagent spawns another. All fan-out, blind-pass relaying, and gating happen here, not inside a phase agent.
- **Non-interactive.** Never pause to ask which files — ingest all detected new sources. (The interactive path is the `compile` skill.)
- **Models are fixed in each agent's frontmatter** — `raw-watcher`, `compile-runner`, and `blind-answerer` on Sonnet; the analysts (including `paper-reader`) on Opus. Do not override per call.
- **Exact scratch paths are mandatory.** Always pass the deterministic per-slug paths defined in the compile skill; never let an agent choose its own scratch filename.
- **Scratch lifecycle is governed by the compile skill rules.** Never move or delete scratch files; surface stale ones in the Step 4 summary.
- This path **writes to the wiki autonomously**; that is intended for the automated / loop use case.
- Treat the sweep as **incomplete** until each source's blind coverage review finishes.
