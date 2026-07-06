# CLAUDE.md

You are the maintainer of this knowledge base. You are responsible for writing, updating, and maintaining all wiki content.
The user is responsible for curating raw source material, exploring ideas, and asking questions.

Communicate with the user and write wiki content in the repository's configured wiki language.
For this repository, communicate with the user in `zh-TW` and write wiki content in `zh-TW`.

---

## Claude Orchestration

Each maintenance phase has a defined owner. Batch-style phases (`coverage-review`, `lint`) run in a fresh, decoupled subagent; interactive phases (`compile`, `query`) run on the main thread so they can talk to the user.

- **compile** — main thread runs the `compile` skill directly as orchestrator (it asks the user which raw files to ingest, then spawns the phase subagents). The main thread never reads the raw source or writes wiki pages itself. Treat `compile` and `coverage-review` as separate phases.
- **coverage-review** — after ingest, default to running the blind review protocol unless the user explicitly asks for `compile-only`: the `coverage-reviewer` subagent generates questions and grades, a fresh `blind-answerer` subagent (which has never seen the raw source) runs each wiki-only answer pass, and the orchestrator relays scratch files between them. Never run the wiki-only validation pass in a context that has read the raw source. For batch compile runs, finish ingest for the whole batch first, then review each source in source order.
- **lint** — when the user asks to lint/audit/health-check the wiki, spawn the project-scoped `lint-runner` subagent. It reports by default and only fixes structural issues when the user explicitly authorizes a fix; surface its Lint Report verbatim.
- **query** — main thread runs the `query` skill directly (it is an interactive question-and-answer flow).

### Automated ingest pipeline (building blocks)

The pipeline has two layers. **`compile` is the canonical per-source pipeline**; **`sweep` is the discovery wrapper** that runs `compile` on every newly detected source.

**`compile` (per source):** the orchestrator (main thread) reads no source content itself; it verifies targets, spawns subagents with **exact scratch paths**, and relays files between phases. Sources are read by the analysts, the writer, and the reviewer directly with the Read tool (PDFs render to page images via `pdftoppm`; see Tooling below).

1. **Analyst team** (Opus 4.8, parallel) — spawn all three in one message, passing the raw source path and the exact findings output path each must use (`.claude/scratch/findings/<slug>-theory.md` / `-derivation.md` / `-experiment.md`):
   - **theory-context-analyst** — problem, method lineage, core idea/assumptions, concepts/entities to page.
   - **derivation-checker** — transcribe & re-derive key equations; flag Error/Assumption/Gap.
   - **experiment-synthesizer** — setup, headline results, and the ablation read of *which component drives the gains*.
   Each writes its findings note to its assigned file and returns only the path + a short gist.
2. **Write** (`compile-runner`, Sonnet) — spawned with the raw path + three findings paths; writes the wiki pages (source summary, concept/entity pages, cluster entrances, log, raw-index), emits the coverage handoff payload, and surfaces claim–evidence conflicts.
3. **Blind coverage review** — orchestrated per source, in source order, after the whole batch finishes ingest:
   - **coverage-reviewer** (referee) gets the handoff payload **plus the source's raw path** (so it can verify exact equations/numbers against the source itself); it writes question + answer-key files to `.claude/scratch/coverage/` and grades, but never answers its own questions.
   - **blind-answerer** (Sonnet, fresh per pass) gets only the question file and answers strictly from `wiki/`; the orchestrator relays its answers file back to the reviewer (initial, regression, and holdout passes each use a fresh blind-answerer).
   - On `Ready: yes`, the reviewer fills `Validated On` in `raw/raw-index.md`. Scratch files are left for the user to clean manually — **never move or delete them**.

**`sweep` (discovery wrapper):**

1. **raw-watcher** (Sonnet) — detect raw files not yet recorded as `done` in `raw/raw-index.md`, register each as `pending` (with a post-append table self-check), and return the handoff file list.
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

- Most raw sources are PDFs. **Read them directly with the Read tool** — it reads PDFs natively by rendering each page to an image via `pdftoppm` (poppler is installed on this machine and on PATH). This is the single, canonical way to read a PDF here; there is no separate text-extraction step and no scratch `.txt`.
  - Reading by page range is supported (and preferred for long PDFs) via the Read tool's `pages` parameter.
  - Because pages are read as images, the model sees equations, tables, and figures in their original layout, but the content is not grep-able. When you need to locate a term across a long PDF, read the relevant page range rather than searching text.
  - If `pdftoppm` is ever missing (poppler uninstalled), the native path fails; reinstall poppler rather than reintroducing a text-extraction workaround.
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
