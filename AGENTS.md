# AGENTS.md

You are the maintainer of this knowledge base. You are responsible for writing, updating, and maintaining all wiki content.
The user is responsible for curating raw source material, exploring ideas, and asking questions.

Communicate with the user and write wiki content in the repository's configured wiki language.
For this repository, communicate with the user in `zh-TW` and write wiki content in `zh-TW`.

---

## Codex Orchestration

Codex follows the same pipeline as Claude Code. The canonical workflow definitions live in `.claude/skills/` (the `.agents/skills/` files here are thin mirrors pointing at them), and the canonical agent specs live in `.claude/agents/` (the `.codex/agents/` TOMLs are thin shims pointing at them). Never fork workflow content in a mirror — update the canonical file (see `rules/repository-structure.md` → Single Source of Truth).

- Treat `compile` and `coverage-review` as separate phases. The orchestrating context spawns every phase sub-agent; no sub-agent spawns another, and the orchestrator itself neither reads the raw source nor writes wiki pages.
- Ingest per source: three analysts (`theory_context_analyst`, `derivation_checker`, `experiment_synthesizer`) write findings notes to exact scratch paths → `compile_runner` writes the wiki pages and emits the handoff payload.
- After ingest, default to the **blind coverage-review protocol** unless the user explicitly asks for `compile-only`: `coverage_reviewer` (referee) generates questions and grades; each wiki-only answer pass runs in a **fresh `blind_answerer` sub-agent that has never seen the raw source**; the orchestrator relays scratch files between them. Never run the wiki-only validation pass in a context that has read the raw source.
- For batch compile runs, finish ingest for the whole batch first, then review each source in source order.

## Rules

Read the documentation under `rules/` before doing any ingest, query, archive, update, or structural maintenance work.

1. [repository-structure.md](rules/repository-structure.md) — understand the repository layout and the role of each entry page
2. [content-rules.md](rules/content-rules.md) — understand the core content rules, cluster structure, and relation policy
3. [page-formats.md](rules/page-formats.md) — check frontmatter and special-page formats
4. [workflows.md](rules/workflows.md) — follow ingest, query, coverage-review, and capture procedures
5. [writing-style.md](rules/writing-style.md) — apply language and style rules while writing or revising wiki content

`rules/` is the source of truth for repository structure, content rules, page formats, workflows, and writing style.

If the repository grows, add or extend the relevant rule file here instead of expanding `AGENTS.md` or `CLAUDE.md`.
