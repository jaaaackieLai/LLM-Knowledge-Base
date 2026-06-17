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

The pipeline has two layers. **`compile` is the canonical per-source pipeline**; **`sweep` is the discovery wrapper** that runs `compile` on every newly detected source.

**`compile` (per source):**

1. **Extract** — PDF: system `python` + `pypdf` → `.claude/scratch/<filename>.txt`. Markdown: read directly.
2. **Analyst team** (Opus 4.8, parallel) — spawn all three in one message, passing the extracted-text path and a findings output path under `.claude/scratch/findings/`:
   - **theory-context-analyst** — problem, method lineage, core idea/assumptions, concepts/entities to page.
   - **derivation-checker** — transcribe & re-derive key equations; flag Error/Assumption/Gap.
   - **experiment-synthesizer** — setup, headline results, and the ablation read of *which component drives the gains*.
   Each writes its findings note to `.claude/scratch/findings/` and returns only the path + a short gist.
3. **Write** — main thread writes wiki pages (source summary, concept/entity pages, cluster entrances, log) using analyst findings.
4. **coverage-reviewer** — spawned by the orchestrator per source, in source order, using the handoff payload **plus the source's extracted-text path** (so it can verify exact equations/numbers against the real raw extraction). On `Ready: yes`, the reviewer fills `Validated On` in `raw/raw-index.md`. Scratch files are left for the user to clean manually — **never move or delete them**.

**`sweep` (discovery wrapper):**

1. **raw-watcher** (Sonnet) — detect raw files not yet recorded as `done` in `raw/raw-index.md`, register each as `pending`, and return the handoff file list.
2. **compile** — for each detected source, run the full compile pipeline above (skip compile's auto-discover step since `raw-watcher` already registered the files).

The `compile` skill is the source of truth for the per-source pipeline details. `sweep` delegates everything except discovery to `compile`.

### Running the pipeline — `sweep` skill + `/loop`

The whole pipeline is packaged as the `sweep` skill (`.claude/skills/sweep/`). It is the orchestrator: invoked on the main thread, it runs Steps 1–4 above and spawns every phase subagent (no subagent spawns another). A tick with no new sources is a cheap no-op; a tick with new sources ingests them and gates on coverage review, **writing to the wiki autonomously**.

Run it once on demand with `/sweep`. To run it unattended, the user sets a recurring loop — **default cadence: every 3 hours**:

```
/loop 3h /sweep
```

Omit the interval (`/loop /sweep`) to let the model self-pace; stop by interrupting the loop. The loop drives the main window, which is the team lead that fans out the analysts and runs the gate.

## Tooling (Windows)

- Most raw sources are PDFs. On this machine the Read tool's native PDF path fails (no `pdftoppm`/poppler), so do not rely on it for ingest.
- Extract PDF text with the **system `python` on PATH** + `pypdf` (the old `~/python_env/AI/Scripts/python.exe` venv no longer exists — do not use it; the system `python`, e.g. `...\Python312\python.exe`, already has `pypdf` installed). Set `PYTHONIOENCODING=utf-8` so CJK text in stdout is not garbled. For long PDFs, extract by page range rather than all at once.
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
