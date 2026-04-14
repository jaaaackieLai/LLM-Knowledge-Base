# LLM Knowledge Base

![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20Linux-lightgrey)
![Format](https://img.shields.io/badge/format-Markdown%20%7C%20Plain%20Text-orange)
![Codebase](https://img.shields.io/badge/codebase-language%20agnostic-blue)

This repository contains a personal knowledge base maintained with LLM assistance, inspired by [Karpathy's LLM Wiki idea](https://x.com/karpathy/status/2039805659525644595).

![Visualize](./.github/assets/Visualize.png)

## Usage
Browse the wiki with [Obsidian](https://obsidian.md/), and operate the repository with Codex or [Claude Code](https://claude.ai/code). 
Maintenance rules are organized under `rules/`.

### **Topic Clusters**
In this repository, a **`cluster`** is an internal navigation partition that helps decide which topic entrance a page belongs to first. It is not meant to be a universal taxonomy, and it's not something another person's knowledge base must copy directly.

For an external reader, the important idea is simply that the knowledge base groups content into a small number of topic entrances, then routes readers from cluster pages down to source, concept, entity, comparison, analysis, and question pages. In day-to-day use, start from `wiki/clusters/` instead of searching the whole graph blindly.

If you need the concrete cluster definitions and decision criteria used in this repository, see [rules/content-rules.md](rules/content-rules.md).

### **Skills**
Here we introduce some self-defined skills in Codex and Claude.

| Workflow | Skill | Purpose |
| --- | --- | --- |
| Ingest          | `compile`         | Turn `raw/` source material into `wiki/sources/` pages, assign clusters, and update related concepts, entities, questions, cluster pages, `overview`, and `log`   |
| Query           | `query`           | Answer from `wiki/` by starting from the relevant cluster entrance, and optionally archive durable answers to `wiki/analyses/`                                    |
| Lint            | `lint`            | Audit structural and graph quality issues such as orphan pages, broken links, missing frontmatter, cluster integrity, overview integrity, and weak relation notes |
| Coverage Review | `coverage-review` | Evaluate whether a source is answerable from `wiki/` alone, repair high-confidence gaps, and validate the repair                                                  |
| Language Config | `language`        | Update the repository's configured wiki language across non-wiki maintenance documents and operational instructions                                               |

### **Directory Structure**

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
│   ├── raw-index.md         # Tracking table for source ingest status
│   └── assets/              # Images and attachments
└── wiki/                    # LLM-maintained wiki
    ├── log.md               # Append-only change log
    ├── overview.md          # Top-level wiki entrance
    ├── clusters/            # Topic cluster entrance pages
    ├── sources/             # Per-source summary pages
    ├── concepts/            # Abstract concept pages
    ├── entities/            # Person / tool / organization pages
    ├── comparisons/         # Comparison pages
    ├── analyses/            # Archived query results
    └── questions/           # Open questions and exploration directions
```
