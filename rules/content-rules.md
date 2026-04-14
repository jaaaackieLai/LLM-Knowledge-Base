# Content Rules

This file defines the wiki's content rules, cluster structure, and relation policy.

## Core Rules

1. Never modify files under `raw/`, except `raw/raw-index.md`.
2. Every wiki content page must have YAML frontmatter, except `wiki/log.md`.
3. Update `wiki/log.md` only when there is a substantive wiki content change. Pure tooling or documentation maintenance does not belong there.
4. Use Obsidian wiki links in content pages: `[[page-name]]`. Do not use wiki links inside `wiki/log.md`.
5. Avoid orphan pages. Every wiki content page should have at least one inbound link.
6. Use kebab-case filenames.
7. Update `raw/raw-index.md` after ingest is complete.
8. Every content page has exactly one primary `cluster`. `source`, `concept`, `entity`, `comparison`, `analysis`, and `question` pages must all declare it.
9. Use `bridges` sparingly. Add a bridge only when the page genuinely spans another cluster and the bridge can be justified in one concrete sentence.
10. Clusters are not a substitute for semantic relations. `cluster` and `bridges` provide navigation; `[[links]]` should still reflect high-confidence meaning.
11. Related-page lists must explain the relation. Use the pattern `[[page-name]] — relation explanation`.
12. Add links only with high confidence. If you cannot justify the relation in one concrete sentence supported by raw material or existing wiki content, do not link it.
13. Distinguish direct relations from extended relations. Direct relations come from explicit source statements, dependencies, comparisons, author/work relations, or question/source relations. Extended relations require supported editorial synthesis already grounded elsewhere in the wiki.
14. Avoid vague relation notes such as `related`, `see also`, or other weak catch-all language.
15. `wiki/log.md` records change events only and must stay outside the semantic graph.

## Cluster Keys

This repository currently uses three primary clusters:

- `self-management-growth` — self-management, self-development, creativity, and individual agency
- `deep-learning-research` — deep learning theory, methods, representation learning, and model analysis
- `agent-engineering-practice` — agent workflows, harness design, engineering practice, and AI tool use in research or development

## Cluster Design

A `cluster` is a navigation partition, not a taxonomy tree and not a substitute for tags. It answers "which entrance should a reader start from?" rather than "what topics does this page loosely touch?"

### **When to Reuse an Existing Cluster**

Default to an existing cluster whenever the page can still be explained clearly by that cluster's core questions and inclusion boundary. If the page has a secondary cross-domain angle, prefer `bridges`, cross-references, or `wiki/questions/` instead of creating a new cluster too early.

### **When to Create a New Cluster**

Create a new cluster only when all of the following are true:

1. A stable new problem space has emerged: at least `3` distinct raw sources already exist, or the area is likely to grow into `6+` source / concept / entity pages.
2. The area has its own inclusion boundary: you can clearly say what belongs in it and what does not, and that boundary is not just a subtopic of an existing cluster.
3. It can function as a genuine entry point: for users asking about that area, the natural first step would be to start from that cluster page.
4. Keeping everything inside existing clusters is causing routing problems: repeated bridge overload, repeated boundary exceptions, or one cluster page being forced to answer two different sets of core questions.

### **When Not to Create a New Cluster**

- A topic with only one source or a short-lived side branch
- A method, lens, workflow, or person that naturally spans multiple clusters
- A need that can be handled by tags or cross-references instead of a new entrance page
- A temporarily dense topic whose core questions still fit an existing cluster

If the situation is ambiguous, keep the content in an existing cluster first and record the tension via `bridges` or `wiki/questions/`. Split out a new cluster only if repeated ingest work keeps exposing the same entrance problem.

### **When a New Cluster Is Created**

1. Add the new cluster key and definition here.
2. Create a matching page under `wiki/clusters/` with clear scope, boundary, and representative pages.
3. Update `wiki/overview.md` so the new cluster becomes a formal entrance.
4. Reassign existing pages only when necessary. Do not reshuffle the whole repository just for symmetry.
