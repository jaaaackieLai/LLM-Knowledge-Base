# Writing Style

This file defines language behavior and editorial style for repository maintenance.

## Language Policy

- For this repository, communicate with the user in `zh-TW` and write wiki content in `zh-TW`.
- For a different knowledge base, replace `zh-TW`, the relation labels, and the examples accordingly.

## Style Guide

- Tone: objective, precise, and concise
- Summary style: conclusion first, then details
- Citations: point back to the underlying raw source or the relevant wiki pages
- Cluster discipline: decide the primary cluster first, then assign a `subcluster` only when one topic entrance is clearly dominant; add `bridges` only when truly necessary
- Related-page notes: every `[[wiki-link]]` in a related section must include a concrete relation explanation
- Relation labels: write explicit relation labels in `zh-TW`, for example `直接關聯：...` and `延伸關聯：...`
- Link threshold: do not create links that cannot be justified by explicit source support or supported editorial synthesis
- Cluster pages: treat them as broad-domain entrances and boundary pages, not as replacement encyclopedia articles
- Subcluster pages: treat them as specific topic entrances that collect the most representative pages for that line
- Log behavior: keep `wiki/log.md` free of wiki links
- Length guidance: source summaries usually `300-800`  words; concept pages usually `500-1500` words
- Math notation: use LaTeX, with `$$...$$` for display math and `$...$` for inline math

## Concreteness and First-Appearance Terms (no naked jargon)

Write concretely, not as a list of name-dropped terms. A first-time reader must be able to follow the page without already knowing the vocabulary.

When a technical term or concept appears for the **first time** on a page, it must resolve one of three ways — never left as bare, unexplained jargon:

1. **Link to an existing page** — if a `wiki/concepts/` or `wiki/entities/` page already covers it, add the `[[...]]` link at first mention.
2. **Inline gloss (+ link)** — for a secondary or minor term, give a one-clause plain-language gloss at first mention (and link if a page exists). Do not create a new page yet.
3. **New concept page** — create a `wiki/concepts/` page only when the term meets the promotion threshold below.

**Promotion threshold (gloss → its own page).** Give a term its own page when at least one holds:

- it is **central** to the source's contribution (the method cannot be understood without it), or
- it is referenced by **two or more sources** already in the wiki (recurring → worth a hub), or
- it already has an open question, or is something a reader would plausibly query on its own.

Otherwise keep it as an inline gloss now and promote it later when it recurs. **Do not create stub pages**: if you cannot write a substantive page (around the concept-page length guidance), use an inline gloss instead of a one-or-two-line placeholder.

This rule directly serves answerability: a reader should grasp every first-appearing term without leaving the page for `raw/`.

**Example (bad → fixed).** Bad — naked jargon a newcomer cannot parse:

> Transformer 的 encoder 與 decoder 都由 self-attention、feed-forward network、residual connection 與 layer normalization 疊成，並加入 positional encoding 表示序列順序。

Fixed — central terms linked to their own pages, minor terms glossed inline:

> Transformer 的 encoder 與 decoder 都由 [[self-attention]]（讓每個位置直接彙整序列中所有位置的資訊）、前饋網路、殘差連接（將輸入直接加到子層輸出以利深層訓練）與層正規化堆疊而成。因為沒有遞迴或卷積，模型以 [[positional-encoding]] 為序列位置注入順序資訊。
