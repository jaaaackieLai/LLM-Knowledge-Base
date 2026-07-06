# Repository Structure

This file describes the repository layout and the role of each top-level maintenance document.

## Directory Structure

```text
second-brain/
├── AGENTS.md                # Codex agent entry file
├── CLAUDE.md                # Claude Code entry file (orchestration + tooling)
├── README.md
├── rules/
│   ├── repository-structure.md
│   ├── content-rules.md
│   ├── page-formats.md
│   ├── workflows.md
│   └── writing-style.md
├── .claude/
│   ├── skills/              # canonical skills: compile, coverage-review, lint, query, sweep, language
│   ├── agents/              # subagent specs: raw-watcher, three analysts, compile-runner, coverage-reviewer, blind-answerer, lint-runner
│   └── scratch/
│       ├── findings/        # analyst findings notes, one set per source (left for manual cleanup)
│       └── coverage/        # coverage-review question / answer-key / blind-answer files (left for manual cleanup)
├── .agents/
│   └── skills/              # Codex skill mirrors — thin pointers to .claude/skills/ (never fork content here)
├── .codex/
│   └── agents/              # Codex sub-agent definitions — thin shims pointing at .claude/agents/ specs
├── docs/
│   └── assets/              # repo-level illustrations, not wiki content
├── raw/                     # Immutable source material
│   ├── raw-index.md         # Tracking table for ingest status
│   └── assets/              # Images and attachments
└── wiki/                    # LLM-maintained wiki
    ├── log.md               # Append-only change log, outside the semantic graph (rotated yearly to log-YYYY.md)
    ├── overview.md          # Top-level wiki entrance, links only to clusters
    ├── clusters/            # Top-level cluster entrance pages
    ├── subclusters/         # Cluster-internal topic entrances
    ├── sources/             # Per-source summary pages
    ├── concepts/            # Abstract concept pages
    ├── entities/            # Person / tool / organization pages
    ├── comparisons/         # Comparison pages
    ├── analyses/            # Archived query results
    └── questions/           # Open questions and exploration directions
```

## Entry Roles

- `AGENTS.md` / `CLAUDE.md`: short entry files that define role, language behavior, orchestration, and where to find the rules
- `wiki/overview.md`: the only top-level wiki entrance page; it routes readers into clusters
- `wiki/clusters/`: entrance pages for each major topic partition; they define the broad domain and route readers to subclusters
- `wiki/subclusters/`: second-level entrance pages for specific topic lines inside a cluster
- `wiki/log.md`: a maintenance log, not part of the semantic graph; rotate yearly per content-rules.md rule 19
- `raw/raw-index.md`: a tracking table for source ingest and validation status, not part of the semantic graph

## Single Source of Truth

- `.claude/skills/` holds the canonical, executable workflow definitions; `rules/workflows.md` holds the harness-agnostic phase contract.
- `.agents/skills/` (Codex skills) and `.codex/agents/` (Codex sub-agents) are thin mirrors that point back at the canonical `.claude/` files. Never edit workflow substance in a mirror — update the canonical file and keep the mirror a pointer, so the two harnesses cannot drift.
