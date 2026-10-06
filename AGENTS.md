# AGENTS.md

You are the maintainer of this knowledge base. You write, update, and maintain all wiki content.
The user curates raw source material, explores ideas, and asks questions.

For this repository, communicate with the user in `zh-TW` and write wiki content in `zh-TW`.

---

## Domains

The knowledge base is domain-first. A domain is a folder directly under the repository root that contains `GUIDE.md`; the folder name is the domain key. Each domain holds its own `raw/` and `wiki/`, and `index.md` at the root lists every domain.

- `rules/` is the shared framework. A domain's `GUIDE.md` overrides content and analysis defaults (analyst team, source-page structure, coverage question types) for that domain only. It never overrides the shared framework (`rules/content-rules.md` → GUIDE.md precedence).
- Every workflow runs inside one domain. Links stay within one domain.
- Never create a domain on your own. Design a new domain's `GUIDE.md` with the user first.

## Orchestration

Claude Code and Codex run the same pipeline. Canonical skills live in `.claude/skills/` and canonical agent specs in `.claude/agents/`. `.agents/skills/` and `.codex/agents/` are thin mirrors: edit only the canonical file (see `rules/repository-structure.md` → Single Source of Truth).

The main thread is the orchestrator. It spawns every phase sub-agent, and no sub-agent spawns another.

- **compile** — the main thread runs the `compile` skill: it asks the user which raw files to ingest, reads the source's `GUIDE.md`, then spawns the domain's analyst team, `compile-runner`, and the blind coverage review. The main thread itself never reads the raw source or writes wiki pages.
- **coverage-review** — runs after every ingest unless the user explicitly asks for `compile-only`. Each wiki-only answer pass runs in a fresh `blind-answerer` that has never seen the raw source.
- **sweep** — the unattended entry point: `raw-watcher` scans every domain and registers new raw files, then the `compile` pipeline runs on each. Default cadence: `/loop 3h /sweep`.
- **query** — the main thread runs the `query` skill, since it is an interactive question-and-answer flow. It starts from `index.md`.
- **lint** — spawn `lint-runner`. It reports by default and fixes structural issues only when the user authorizes a fix. Surface its Lint Report verbatim.

## Rules

Read `rules/` before any ingest, query, archive, update, or structural maintenance work, and read the domain's `GUIDE.md` before working inside that domain. `rules/` is the source of truth for repository structure, content rules, page formats, workflows, and writing style.

1. [repository-structure.md](rules/repository-structure.md) — repository layout, domain identification, and single source of truth
2. [content-rules.md](rules/content-rules.md) — content rules, domains, GUIDE.md precedence, links, and relation policy
3. [page-formats.md](rules/page-formats.md) — frontmatter, `index.md`, `GUIDE.md`, and special-page formats
4. [workflows.md](rules/workflows.md) — phase contract for ingest, coverage review, query, and capture
5. [writing-style.md](rules/writing-style.md) — reply format and wiki writing style

When the repository grows, extend the relevant rule file instead of this one.
