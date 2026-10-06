# Page Formats

## Frontmatter Format

```yaml
---
title: Page Title
type: overview | source | concept | entity | comparison | analysis | question | subcluster
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [raw/filename.md] # path relative to the domain folder; overview / subcluster may use []
subcluster: subcluster-key # optional and singular for source / concept / entity / comparison / analysis / question. Valid keys are defined in <domain>/GUIDE.md → Subcluster Keys
tags: [tag1, tag2]
---
```

The domain is the folder that holds the page, so frontmatter has no domain field.

## `index.md`

`index.md` sits at the repository root. It is the top-level entrance and lists every domain, one line each. It has no frontmatter and stays outside every domain's semantic graph. It uses path links, because every domain has a file named `overview.md`.

```markdown
# Knowledge Base Index

## Domains
- [[deep-learning-research/wiki/overview|deep-learning-research]] — one-line scope from GUIDE.md
```

## `<domain>/GUIDE.md`

`GUIDE.md` defines one domain. Git tracks it. It has no frontmatter. Precedence rules are in `content-rules.md` → GUIDE.md precedence.

```markdown
# <domain-key>

## Scope
- 收：...
- 不收：...

## Material
type: paper | other
One sentence on what the source material is.

## Analysts
<!-- optional; when omitted, the default team runs -->
- theory-context-analyst → <slug>-theory.md
- derivation-checker → <slug>-derivation.md
- paper-reader → <slug>-paper.md

## Source Page
<!-- optional; when omitted, the compile skill W2 default structure applies -->

## Coverage Questions
<!-- optional; when omitted, the coverage-review skill Step 2 default question types apply -->

## Subcluster Keys
- `key` — definition
```

- `## Analysts` lists one analyst per line as `<agent-name> → <slug>-<suffix>.md`. The orchestrator passes each analyst the exact findings path `.claude/scratch/findings/<slug>-<suffix>.md`, and the analyst writes its note there.
- `## Subcluster Keys` is the single source of truth for that domain's valid `subcluster` values.

## `<domain>/wiki/overview.md`

`overview.md` is the domain entrance. It routes readers to subclusters first, then states the scope boundary. It does not list regular content pages one by one. The 收錄邊界 section restates `GUIDE.md` → Scope for readers and must stay consistent with it.

```yaml
---
title: 領域：領域名稱
type: overview
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: []
tags: [overview, navigation]
---
```

```markdown
# 領域：領域名稱

## 核心問題
- ...

## 收錄邊界
- 收：...
- 不收：...

## 子聚落
- [[subcluster-example]] — specific topic entrance
```

## `<domain>/wiki/subclusters/*.md`

Subcluster pages are specific topic entrances inside one domain.

```yaml
---
title: 子聚落：主題名稱
type: subcluster
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: []
tags: [subcluster, navigation]
---
```

```markdown
# 子聚落：主題名稱

## 核心問題
- ...

## 收錄邊界
- 收：...
- 不收：...

## 代表性頁面
- [[source-example]] — source relation
- [[concept-example]] — concept relation
```

## `<domain>/raw/raw-index.md`

Each domain has one `raw-index.md`. It tracks ingest and validation status for that domain's raw source material. It does not participate in the semantic graph.

```markdown
# Raw Sources Index

| File | Status | Ingested On | Validated On | Source Page | Notes |
|------|--------|-------------|--------------|-------------|-------|
| example.md | done | 2026-04-06 | 2026-04-11 | `source-example` | - |
| pending.md | pending | - | - | - | waiting for ingest |
```

Status values: `pending` / `done` / `update-needed`

Formatting:

- The File cell holds the filename under `<domain>/raw/`, with no folder prefix.
- One row per file on disk, written as one compact line with no alignment padding. Padded tables make anchored edits fragile.
- Each cell holds one short phrase. Long validation narratives belong in `<domain>/wiki/log.md`.
- Escape a literal `|` inside a cell as `\|`, otherwise the row splits.
