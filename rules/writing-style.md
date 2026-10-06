# Writing Style

The configured languages are set in `AGENTS.md`.

## Chat and Terminal Replies

- Math: write equations in Unicode (e.g. `γ = 1/c`, `∑`, `√`, `≤`, `⊙`). The terminal shows `$...$` as literal source.
- Links: reference wiki pages as Markdown links with repo-relative paths, e.g. `[contrastive-learning](deep-learning-research/wiki/concepts/contrastive-learning.md)`, so the user can ctrl+click to the file. This covers sub-agent reports relayed to the user. `[[wiki-links]]` belong in wiki files only, because chat renders them as plain text.

## Wiki Content

- Tone: objective, precise, and concise. Conclusion first, then details.
- Citations: point back to the underlying raw source or the relevant wiki pages.
- Math: LaTeX, with `$$...$$` for display math and `$...$` for inline math. Obsidian renders it.
- Length: source summaries usually `500-1300` 字. Concept pages usually `800-2400` 字.

## First-Appearance Terms (no naked jargon)

Write concretely, not as a list of name-dropped terms. A first-time reader must be able to follow the page without already knowing the vocabulary.

When a technical term or concept appears for the **first time** on a page, resolve it one of three ways:

1. **Link to an existing page.** If a `<domain>/wiki/concepts/` or `<domain>/wiki/entities/` page in the same domain already covers it, add the `[[...]]` link at first mention.
2. **Inline gloss (+ link).** For a secondary or minor term, give a one-clause plain-language gloss at first mention, and link it if a page exists. Do not create a new page yet.
3. **New concept page.** Create a `<domain>/wiki/concepts/` page only when the term meets the promotion threshold below.

**Promotion threshold (gloss → its own page).** Give a term its own page when at least one holds:

- it is **central** to the source's contribution (the method cannot be understood without it), or
- it is referenced by **two or more sources** already in the wiki, or
- it already has an open question, or a reader would plausibly query it on its own.

Otherwise keep the inline gloss and promote the term later when it recurs. A new page must be substantive, near the concept-page length above. When you cannot write that much, use an inline gloss instead of a one-or-two-line stub.

This rule serves answerability: a reader grasps every first-appearing term without leaving the page for `<domain>/raw/`.

**Example (bad → fixed).** Bad, naked jargon a newcomer cannot parse:

> Transformer 的 encoder 與 decoder 都由 self-attention、feed-forward network、residual connection 與 layer normalization 疊成，並加入 positional encoding 表示序列順序。

Fixed, central terms linked to their own pages and minor terms glossed inline:

> Transformer 的 encoder 與 decoder 都由 [[self-attention]]（讓每個位置直接彙整序列中所有位置的資訊）、前饋網路、殘差連接（將輸入直接加到子層輸出以利深層訓練）與層正規化堆疊而成。因為沒有遞迴或卷積，模型以 [[positional-encoding]] 為序列位置注入順序資訊。
