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
| Ingest (interactive) | `compile`    | Turn `raw/` source material into `wiki/sources/` pages, assign clusters, and update related concepts, entities, questions, cluster pages, `overview`, and `log`   |
| Ingest (automated)   | `sweep`      | Orchestrate the non-interactive ingest pipeline end-to-end — detect new `raw/` sources, run the analyst team, write the wiki pages, and gate on coverage review (Claude Code only)   |
| Query           | `query`           | Answer from `wiki/` by starting from the relevant cluster entrance, and optionally archive durable answers to `wiki/analyses/`                                    |
| Lint            | `lint`            | Audit structural and graph quality issues such as orphan pages, broken links, missing frontmatter, cluster integrity, overview integrity, and weak relation notes |
| Coverage Review | `coverage-review` | Evaluate whether a source is answerable from `wiki/` alone, repair high-confidence gaps, and validate the repair                                                  |
| Language Config | `language`        | Update the repository's configured wiki language across non-wiki maintenance documents and operational instructions                                               |

### **Ingesting sources: interactive vs automated**

There are two ways to turn `raw/` material into wiki pages — same destination, different driver:

- **Interactive — `/compile`** (Codex or Claude Code). Runs on the main thread and asks which `raw/` files to ingest, writing pages with you in the loop. Best for ad-hoc, one-off ingest you want to steer.
- **Automated / non-interactive — `/sweep`** (Claude Code). One command runs the entire pipeline without prompting: it finds every `raw/` source not yet marked `done` in [raw/raw-index.md](raw/raw-index.md), and for each one extracts the text, fans out a three-analyst team (theory context, derivation audit, experiment synthesis), writes the wiki pages, then gates on an **independent coverage review** before marking the source validated.
  - On demand: `/sweep`
  - Unattended on a schedule (default every 3 hours): `/loop 3h /sweep`. Omit the interval (`/loop /sweep`) to let the model self-pace; stop by interrupting the loop.

The automated path is an orchestrator that chains single-responsibility subagents in `.claude/agents/` (no subagent spawns another):

```text
raw-watcher → (per source) extract → {theory-context-analyst, derivation-checker, experiment-synthesizer} → compile-runner → coverage-reviewer
```

PDF text is extracted once with the system `python` (+ `pypdf`) into `.claude/scratch/`; the analysts write findings notes to `.claude/scratch/findings/`, which the writer (`compile-runner`) reads. The coverage reviewer's `Ready: yes/no` verdict gates completion and fills the source's validation date in `raw/raw-index.md`. Scratch files are left in place for you to clean manually (the repository never deletes files). See [CLAUDE.md](CLAUDE.md) for the full orchestration spec.

> The automated pipeline (`sweep` + the subagent team) is Claude Code-specific, since it relies on subagents. The `compile`, `query`, `lint`, `coverage-review`, and `language` skills work in both Codex and Claude Code.

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
│   └── skills/              # Codex skills (compile, query, lint, coverage-review, language)
├── .claude/
│   ├── agents/              # Claude Code subagents (analyst team, compile-runner, coverage-reviewer, raw-watcher, lint-runner)
│   ├── skills/              # Claude Code skills (the above + sweep)
│   └── scratch/             # Transient PDF extractions & analyst findings (findings/); user-cleaned
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
