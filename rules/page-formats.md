# Page Formats

This file defines frontmatter requirements and the expected format of special navigation and tracking pages.

## Frontmatter Format

```yaml
---
title: Page Title
type: overview | source | concept | entity | comparison | analysis | question | cluster | subcluster
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [raw/filename.md] # overview / cluster may use []
cluster: self-management-growth | deep-learning-research | agent-engineering-practice # required for source / concept / entity / comparison / analysis / question / subcluster; omit for overview / cluster
subcluster: information-bottleneck-and-mutual-information # optional for source / concept / entity / comparison / analysis / question
bridges: [other-cluster-key] # optional; usually omitted for overview / cluster / subcluster
tags: [tag1, tag2]
---
```

## `wiki/overview.md`

`wiki/overview.md` is the only top-level navigation page. It should list cluster entrances only, not regular content pages or subcluster pages.

```markdown
# Knowledge Base Overview

## Clusters
- [[cluster-self-management-growth]] — cluster entrance
- [[cluster-deep-learning-research]] — cluster entrance
- [[cluster-agent-engineering-practice]] — cluster entrance
```

## `wiki/clusters/*.md`

Cluster pages are broad-domain entrances. They should route readers to subclusters first, then explain boundary and bridge routes.

```markdown
# 聚落：領域名稱

## 核心問題
- ...

## 收錄邊界
- 收：...
- 不收：...

## 子聚落
- [[subcluster-example]] — specific topic entrance
```

## `wiki/subclusters/*.md`

Subcluster pages are specific topic entrances inside one parent cluster.

```yaml
---
title: 子聚落：主題名稱
type: subcluster
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: []
cluster: deep-learning-research
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
