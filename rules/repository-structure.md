# Repository Structure

This file describes the repository layout and the role of each top-level maintenance document.

## Directory Structure

```text
llm-knowledge-base/
├── AGENTS.md                # Codex agent entry file
├── CLAUDE.md                # Claude Code entry file
├── rules/
│   ├── README.md            # Documentation index
│   ├── repository-structure.md
│   ├── content-rules.md
│   ├── page-formats.md
│   ├── workflows.md
│   └── writing-style.md
├── .agents/
│   └── skills/              # Codex skills
├── .claude/
│   └── commands/            # Claude Code slash commands
├── raw/                     # Immutable source material
│   ├── raw-index.md         # Tracking table for ingest status
│   └── assets/              # Images and attachments
└── wiki/                    # LLM-maintained wiki
    ├── log.md               # Append-only change log, outside the semantic graph
    ├── overview.md          # Top-level wiki entrance, links only to clusters
    ├── clusters/            # Topic-cluster entrance pages
    ├── sources/             # Per-source summary pages
    ├── concepts/            # Abstract concept pages
    ├── entities/            # Person / tool / organization pages
    ├── comparisons/         # Comparison pages
    ├── analyses/            # Archived query results
    └── questions/           # Open questions and exploration directions
```

## Entry Roles

- `AGENTS.md` / `CLAUDE.md`: short entry files that define role, language behavior, and where to find the rules
- `rules/README.md`: the documentation index; start here when you are unsure which rule file applies
- `wiki/overview.md`: the only top-level wiki entrance page; it routes readers into clusters
- `wiki/clusters/`: entrance pages for each major topic partition, including boundaries and bridge routes
- `wiki/log.md`: a maintenance log, not part of the semantic graph
- `raw/raw-index.md`: a tracking table for source ingest and validation status, not part of the semantic graph
