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
8. Every non-navigation content page has exactly one primary `cluster`. `source`, `concept`, `entity`, `comparison`, `analysis`, and `question` pages must all declare it.
9. `subcluster` is optional but singular. Use it when the page clearly belongs to one stable topic entrance inside its parent cluster.
10. `cluster` and `subcluster` are navigation aids, not a substitute for semantic relations. `[[links]]` should still reflect high-confidence meaning.
11. Use `bridges` sparingly. Add a bridge only when the page genuinely spans another cluster and the bridge can be justified in one concrete sentence.
12. Related-page lists must explain the relation. Use the pattern `[[page-name]] — relation explanation`.
13. Add links only with high confidence. If you cannot justify the relation in one concrete sentence supported by raw material or existing wiki content, do not link it.
14. Distinguish direct relations from extended relations. Direct relations come from explicit source statements, dependencies, comparisons, author/work relations, or question/source relations. Extended relations require supported editorial synthesis already grounded elsewhere in the wiki.
15. Avoid vague relation notes such as `related`, `see also`, or other weak catch-all language.
16. `wiki/log.md` records change events only and must stay outside the semantic graph.
17. No naked jargon. A technical term's first appearance on a page must resolve to an existing `[[link]]`, an inline gloss, or a new concept page, following the tiered first-appearance-term policy in `writing-style.md`. Do not create stub pages just to satisfy a link.
18. Source page filenames are topic-oriented: name a `wiki/sources/` page after the paper's core method/concept in kebab-case, with no venue or year prefix. Example: `source-graph-information-bottleneck` (✅), not `source-neurips-2020-graph-information-bottleneck` (❌). The reader should recognize the topic from the filename without needing the venue.

## Cluster Keys

This repository currently uses four primary clusters:

- `self-management-growth` — self-management, self-development, creativity, and individual agency
- `deep-learning-research` — deep learning theory, methods, representation learning, and model analysis
- `agent-engineering-practice` — agent workflows, harness design, engineering practice, and AI tool use in research or development
- `quant-finance-interview-prep` — quantitative finance interview preparation: probability/statistics, regression/data-science, market making, and firm-specific question banks

## Subcluster Keys

Current subclusters are grouped under their parent cluster:

### `self-management-growth`

- `identity-and-agency` — identity transition, agency recovery, long-term direction, and dispersion control
- `multi-interest-integration` — organizing multiple interests into a coherent knowledge or work trajectory
- `creative-recovery` — rebuilding creative capacity by reducing overload, restoring boredom, and re-entering sustained making

### `deep-learning-research`

- `information-bottleneck-and-mutual-information` — IB, MI estimation, variational bounds, and information-theoretic representation learning
- `contrastive-representation-learning` — contrastive objectives, view design, supervised/self-supervised variants, and multimodal contrastive pretraining
- `time-series-modeling-and-explainability` — time-series forecasting, time-series explanations, and sequence-specific interpretation methods
- `transformers-and-model-analysis` — Transformer mechanisms, attention structure, attribution, and mechanistic or geometric model analysis
- `weight-space-learning-and-parameter-generation` — model weight space learning, model zoo representations, permutation-aware alignment, and neural network parameter generation
- `generative-dynamics-and-scaling` — generative modeling, world models, dynamical systems, posterior sampling, and scaling behavior

### `agent-engineering-practice`

- `agent-workflows-and-harnesses` — harness design, workflow discipline, living specs, and operational control surfaces
- `human-judgment-and-ai-collaboration` — cognitive outsourcing, AI fatigue, supervision, pedagogy, and judgment-preserving collaboration
- `technical-communication-and-research-writing` — code comments, technical exposition, and research argument structure

## Cluster and Subcluster Design

A `cluster` is the broad navigation layer for the second brain. It answers "which large domain am I in?" A `subcluster` is the more specific entrance layer inside that domain. It answers "which concrete topic line should I enter first?"

In this repository, `cluster` is intentionally broader than `subcluster`:

- `cluster` preserves large-domain adjacency, so readers can still see how neighboring themes belong to the same map region
- `subcluster` provides the sharper entrance for a stable topic that would be too blurred if left only at cluster level

### **When to Reuse an Existing Subcluster**

Default to an existing subcluster whenever the page clearly fits one established topic line inside the parent cluster. If the page has a secondary angle, prefer ordinary `[[links]]`, relation notes, or `bridges` instead of splitting entrances too early.

### **When to Create a New Subcluster**

Create a new subcluster when all of the following are true:

1. A specific topic line has become stable: at least `3` strongly related pages already exist, or more are clearly expected soon.
2. The topic has a clear entrance question and boundary inside the parent cluster.
3. Readers would benefit from entering through that topic page rather than scanning a long flat cluster page.
4. The topic is still part of the parent cluster's larger domain and does not need a new top-level map region.

### **When to Create a New Cluster**

Create a new cluster only when all of the following are true:

1. A stable new large-domain problem space has emerged and it no longer feels like a subtopic of an existing cluster.
2. The new area needs to sit beside existing clusters in the top-level map, not under one of them.
3. Readers would lose important domain adjacency if it stayed buried as only a subcluster.
4. Existing clusters are being forced to mix fundamentally different map regions instead of neighboring topic lines.

### **When Not to Create a New Cluster**

- A topic with only one source or a short-lived side branch
- A topic that is specific and stable, but still clearly belongs under an existing cluster as a subcluster
- A method, lens, workflow, or person that naturally spans multiple clusters
- A need that can be handled by tags, links, or cross-references instead of a new entrance page
- A temporarily dense topic whose core questions still fit an existing cluster's map region

If the situation is ambiguous, keep the existing `cluster`, try an existing `subcluster` first, or leave `subcluster` empty until a stable topic entrance actually emerges.

### **When a New Subcluster Is Created**

1. Add the new subcluster key and definition here.
2. Create a matching page under `wiki/subclusters/` with clear scope, boundary, and representative pages.
3. Update the parent page under `wiki/clusters/` so the subcluster becomes part of the formal entrance layer.
4. Reassign existing pages only when helpful. Do not reshuffle the whole repository just for symmetry.

### **When a New Cluster Is Created**

1. Add the new cluster key and definition here.
2. Create a matching page under `wiki/clusters/`.
3. Update `wiki/overview.md` so the new cluster becomes a formal entrance.
4. Add or move subclusters only where needed. Do not reshuffle the whole repository just for symmetry.
