---
name: sweep
description: Orchestrate the unattended ingest pipeline end to end — detect new raw sources, run the analyst team, write the wiki, and gate on coverage review. Use this as the single entry point for automated/batch ingest, typically driven by /loop. It is the orchestrator (team lead) that spawns every phase subagent; no phase agent spawns another.
---

# Ingest Sweep (Automated Pipeline Orchestrator)

One-command orchestration of the automated ingest path. This skill runs on the **main thread** so it can spawn subagents (subagents cannot). It is the **team lead**: it fans out the analyst team and runs the coverage gate itself — no phase agent spawns another.

## When to Use

- Unattended / batch ingest of whatever new sources have appeared under `raw/`
- Driven by `/loop` on an interval (see `CLAUDE.md` for the loop command)
- Any time you want "scan raw, compile everything new, and validate" in one step

For ad-hoc, user-driven ingest where you want to choose files interactively, use the `compile` skill instead.

## Input

- No file argument needed — the sweep discovers new sources itself via `raw-watcher`.
- **Non-interactive**: it ingests all detected new sources without asking which ones.
- Default: allow coverage-review self-heal (do not pass review-only) unless the invoker explicitly says review-only.

## Workflow

### Step 1: Detect (raw-watcher)

- Spawn the `raw-watcher` subagent (Agent tool, `subagent_type: raw-watcher`).
- It registers untracked files into `raw/raw-index.md` as `pending` and returns a Verdict with the handoff file list (newly registered + already pending).

### Step 2: Gate on new sources

- If the Verdict is `New sources: no` → report "nothing to ingest" and stop. This is the common, cheap tick.
- If `New sources: yes (N)` → take the N-file list and continue.

### Step 3: Ingest each source (finish the whole batch first)

For each source in the list, in source order:

1. **Extract once.** If `.pdf`, extract text with the user venv (`~/python_env/AI/Scripts/python.exe` + `pypdf`, `PYTHONIOENCODING=utf-8`, by page range for long PDFs) to a scratch path `.claude/scratch/<filename>.txt`. If `.md`, use the raw path directly (no extraction). Extracting once here avoids each analyst and the writer re-extracting the same PDF.
2. **Fan out the analyst team (parallel).** In a single message, spawn all three read-only analysts, passing both the raw path and the extracted-text path:
   - `theory-context-analyst`
   - `derivation-checker`
   - `experiment-synthesizer`

   Collect their three findings notes.
3. **Write (compile-runner).** Spawn `compile-runner` with the source path, the extracted-text path, and the three findings notes. It writes the wiki pages (compile Steps 2–6), enforces the no-naked-jargon rule, and returns a coverage-review handoff payload. It does **not** run the gate.

Complete Step 3 for **every** source before moving on (batch rule inherited from the `compile` skill).

### Step 4: Coverage gate (coverage-reviewer), per source in source order

- For each ingested source, spawn a fresh `coverage-reviewer` with that source's handoff payload **and the source's pre-extracted text path** (`.claude/scratch/<source>.txt`) — so the reviewer can verify exact equations/numbers against the real raw extraction (the native PDF Read path fails here) instead of trusting only the writer's transcription.
- Relay each reviewer's Report section **verbatim**; do not regrade or rewrite the verdict.
- The reviewer fills `Validated On` on pass, or leaves it blank with an unresolved-gap note on fail.
- **Evict scratch on pass.** Once a source's review returns `Ready: yes`, the orchestrator moves that source's `.claude/scratch/<source>.txt` into the OS temp directory (e.g. `mv` it under `"$(cygpath -u "$TEMP" 2>/dev/null || echo /tmp)/second-brain-scratch/"` — the `/tmp` fallback is used when `cygpath`/`$TEMP` are unavailable, as in this machine's Git Bash). On `Ready: no` / unresolved gap, leave it in `.claude/scratch/` for debugging. The reviewer never moves it (it has no Bash) — eviction is the orchestrator's job.

### Step 5: Summary

- Report per source: ingested (source page) + coverage verdict (`Ready: yes/no`).
- List anything skipped (extraction failure, unreadable source, etc.).

## Rules

- **Orchestrator-only spawning.** This skill spawns every subagent; no subagent spawns another. The analyst fan-out and the coverage gate live here, not inside a phase agent.
- **Non-interactive.** Never pause to ask which files — ingest all detected new sources. (The interactive path is the `compile` skill.)
- **Models are fixed in each agent's frontmatter** — `raw-watcher` and `compile-runner` on Sonnet; the three analysts on Opus. Do not override per call.
- **Scratch lifecycle & recycling (repo → temp on pass).** The orchestrator creates/overwrites `.claude/scratch/<source>.txt` before the analyst fan-out; it stays in-repo (inspectable) through the analysts, writer, and coverage review. **After that source's review passes (`Ready: yes`), the orchestrator moves the file out to the OS temp directory** (`second-brain-scratch/` under `$TEMP`), so the repo stays clean and the OS reclaims it. On a failed review it is **left in `.claude/scratch/` for debugging**. No agent ever *deletes* scratch (repo deletion policy); the only motion is the orchestrator's post-pass `mv` to temp, and only the orchestrator has Bash to do it. Filenames are deterministic per source, so any re-run before eviction overwrites rather than accumulates.
- This path **writes to the wiki autonomously**; that is intended for the automated / loop use case.
- Treat the sweep as **incomplete** until each source's coverage review finishes.
