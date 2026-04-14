# Page Formats

This file defines frontmatter requirements and the expected format of special navigation and tracking pages.

## Frontmatter Format

```yaml
---
title: Page Title
type: overview | source | concept | entity | comparison | analysis | question | cluster
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [raw/filename.md] # overview / cluster may use []
cluster: self-management-growth | deep-learning-research | agent-engineering-practice # omit for overview / cluster
bridges: [other-cluster-key] # optional; usually omitted for overview / cluster
tags: [tag1, tag2]
---
```

## `wiki/overview.md`

`wiki/overview.md` is the only top-level navigation page. It should list cluster entrances only, not regular content pages.

```markdown
# Knowledge Base Overview

## Clusters
- [[cluster-self-management-growth]] — cluster entrance
- [[cluster-deep-learning-research]] — cluster entrance
- [[cluster-agent-engineering-practice]] — cluster entrance
```

## `raw/raw-index.md`

`raw/raw-index.md` tracks ingest and validation status for raw source material. It does not participate in the semantic graph.

```markdown
# Raw Sources Index

| File | Status | Ingested On | Validated On | Source Page | Notes |
|------|--------|-------------|--------------|-------------|-------|
| example.md | done | 2026-04-06 | 2026-04-11 | `source-example` | - |
| pending.md | pending | - | - | - | waiting for ingest |
```

Allowed status values: `pending` / `in-progress` / `done` / `update-needed`
