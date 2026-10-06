# Repository Structure

## Directory Structure

```text
LLM-Knowledge-Base/
├── AGENTS.md                # Entry file for Claude Code and Codex: role, language, orchestration, pointer to rules/
├── README.md
├── index.md                 # Top-level entrance: lists every domain with a path link to its overview
├── rules/                   # Shared framework: structure, content, formats, workflows, and style
├── .claude/
│   ├── skills/              # Canonical skills: compile, coverage-review, lint, query, sweep, language
│   ├── agents/              # Canonical sub-agent specs: raw-watcher, analysts (incl. paper-reader), compile-runner, coverage-reviewer, blind-answerer, lint-runner
│   └── scratch/
│       ├── findings/        # Analyst findings notes, one set per source (the user cleans these manually)
│       ├── coverage/        # Coverage-review question, answer-key, and blind-answer files (the user cleans these manually)
│       └── code/            # Official-code clones made by paper-reader (the user cleans these manually)
├── .agents/
│   └── skills/              # Codex skill mirrors
├── .codex/
│   └── agents/              # Codex sub-agent shims
├── .github/
│   └── assets/              # README illustrations, not wiki content
└── <domain>/                # One folder per domain, e.g. deep-learning-research/
    ├── GUIDE.md             # Domain guide: scope, material, analysts, overrides, subcluster keys (tracked by git)
    ├── raw/                 # Immutable source material (gitignored)
    │   ├── raw-index.md     # Ingest and validation status table, outside the semantic graph
    │   └── assets/          # Images and attachments
    └── wiki/                # LLM-maintained wiki for this domain (gitignored)
        ├── overview.md      # Domain entrance: core question, scope boundary, subclusters
        ├── log.md           # Append-only change log, outside the semantic graph
        ├── subclusters/     # Topic entrances inside this domain
        ├── sources/         # Per-source summary pages
        ├── concepts/        # Abstract concept pages
        ├── entities/        # Person / tool / organization pages
        ├── comparisons/     # Comparison pages
        ├── analyses/        # Archived query results
        └── questions/       # Open questions and exploration directions
```

`.gitignore` ignores every folder named `raw/` or `wiki/` at any depth. Git tracks `index.md` and each `<domain>/GUIDE.md`.

## Identifying Domains

- A domain is a folder directly under the repository root that contains `GUIDE.md`. Find domains with the glob `*/GUIDE.md`.
- The folder name is the domain key, in kebab-case.
- A root folder without `GUIDE.md` is not a domain. Skills and agents skip it.
- In rules, skills, and agents, `<domain>` stands for the domain key. Example: `<domain>/wiki/overview.md` resolves to `deep-learning-research/wiki/overview.md`.

## Single Source of Truth

- `.claude/skills/` holds the executable workflow definitions. `rules/workflows.md` holds the harness-agnostic phase contract.
- `rules/` holds the shared framework. Each `<domain>/GUIDE.md` overrides content and analysis defaults for its domain only (`content-rules.md` → GUIDE.md precedence).
- `.agents/skills/` and `.codex/agents/` are thin mirrors that point back at the canonical `.claude/` files. Change workflow substance only in the canonical file and keep each mirror a pointer, so the two harnesses cannot drift.
