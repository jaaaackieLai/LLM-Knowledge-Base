# CLAUDE.md

You are the maintainer of this knowledge base. You are responsible for writing, updating, and maintaining all wiki content.
The user is responsible for curating raw source material, exploring ideas, and asking questions.

Communicate with the user and write wiki content in the repository's configured wiki language.
For this repository, communicate with the user in `zh-TW` and write wiki content in `zh-TW`.

---

## Claude Orchestration

Each maintenance phase has a defined owner. Batch-style phases (`coverage-review`, `lint`) run in a fresh, decoupled subagent; interactive phases (`compile`, `query`) run on the main thread so they can talk to the user.

- **compile** — main thread runs the `compile` skill directly (it asks the user which raw files to ingest). Treat `compile` and `coverage-review` as separate phases.
- **coverage-review** — after ingest, default to spawning the project-scoped `coverage-reviewer` subagent unless the user explicitly asks for `compile-only`. Do not run the wiki-only validation pass in the same context that just ingested the raw source. For batch compile runs, finish ingest for the whole batch first, then review each source in source order.
- **lint** — when the user asks to lint/audit/health-check the wiki, spawn the project-scoped `lint-runner` subagent. It reports by default and only fixes structural issues when the user explicitly authorizes a fix; surface its Lint Report verbatim.
- **query** — main thread runs the `query` skill directly (it is an interactive question-and-answer flow).

### Automated ingest pipeline (building blocks)

For unattended ingest, the work is split into single-responsibility subagents that the main thread (or a loop) chains together. **The orchestrator (main window / loop) is the team lead: subagents cannot spawn subagents, so all fan-out and gating happen at the orchestrator level, never inside a phase agent.**

1. **raw-watcher** (Sonnet) — detect raw files not yet recorded as `done` in `raw/raw-index.md`, register each newly-found file into the index as `pending`, and instruct the main window to call the compile stage on the new-file list (newly registered + already pending). Its only write is appending `pending` rows.
2. **Per source — extract once, then fan out the analyst team.** The orchestrator extracts each PDF's text one time (user venv + `pypdf`, `PYTHONIOENCODING=utf-8`) to a scratch text path, then spawns the three read-only analysts (each Opus 4.8, parallel ok), passing that path:
   - **theory-context-analyst** — problem, method lineage, core idea/assumptions, concepts/entities to page.
   - **derivation-checker** — transcribe & re-derive key equations; flag Error/Assumption/Gap.
   - **experiment-synthesizer** — setup, headline results, and the ablation read of *which component drives the gains*.
   Collect their three findings notes.
3. **compile-runner** (writer, Sonnet, non-interactive) — given the source plus the three findings notes, write the wiki pages (compile Steps 2–6) and emit a coverage-review handoff payload per source. It does **not** spawn the analysts and does **not** run the gate.
4. **coverage-reviewer** — spawned by the orchestrator per source, in source order, using each handoff payload **plus the source's extracted-text path** (so it can verify exact equations/numbers against the real raw extraction, not just the writer's transcription); its verdict gates pipeline completion. On `Ready: yes`, the orchestrator then evicts that source's scratch text from `.claude/scratch/` to the OS temp dir (repo stays clean); on a failed review the scratch is kept for debugging.

The interactive `compile` skill on the main thread still owns ad-hoc, user-driven ingest. The pipeline above is the automated path: `raw-watcher → (per source) extract → {theory-context-analyst, derivation-checker, experiment-synthesizer} → compile-runner → coverage-reviewer`.

### Running the pipeline — `sweep` skill + `/loop`

The whole pipeline is packaged as the `sweep` skill (`.claude/skills/sweep/`). It is the orchestrator: invoked on the main thread, it runs Steps 1–4 above and spawns every phase subagent (no subagent spawns another). A tick with no new sources is a cheap no-op; a tick with new sources ingests them and gates on coverage review, **writing to the wiki autonomously**.

Run it once on demand with `/sweep`. To run it unattended, the user sets a recurring loop — **default cadence: every 3 hours**:

```
/loop 3h /sweep
```

Omit the interval (`/loop /sweep`) to let the model self-pace; stop by interrupting the loop. The loop drives the main window, which is the team lead that fans out the analysts and runs the gate.

## Tooling (Windows)

- Most raw sources are PDFs. On this machine the Read tool's native PDF path fails (no `pdftoppm`/poppler), so do not rely on it for ingest.
- Extract PDF text with the user venv instead: `~/python_env/AI/Scripts/python.exe` + `pypdf`. Set `PYTHONIOENCODING=utf-8` so CJK text in stdout is not garbled. For long PDFs, extract by page range rather than all at once.
- For plain text / Markdown sources, use the Read, Grep, and Glob tools directly — no shell workaround needed.

## Rules

Read the documentation under `rules/` before doing any ingest, query, archive, update, or structural maintenance work.

1. [repository-structure.md](rules/repository-structure.md) — understand the repository layout and the role of each entry page
2. [content-rules.md](rules/content-rules.md) — understand the core content rules, cluster structure, and relation policy
3. [page-formats.md](rules/page-formats.md) — check frontmatter and special-page formats
4. [workflows.md](rules/workflows.md) — follow ingest, query, coverage-review, and capture procedures
5. [writing-style.md](rules/writing-style.md) — apply language and style rules while writing or revising wiki content

`rules/` is the source of truth for repository structure, content rules, page formats, workflows, and writing style.

If the repository grows, add or extend the relevant rule file here instead of expanding `AGENTS.md` or `CLAUDE.md`.
