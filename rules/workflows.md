# Workflows

This file defines the operational workflows for ingest, query, coverage review, and question capture.

## Ingest

Treat ingest and coverage review as separate phases. Ingest creates or updates wiki content from `raw/`. Coverage review validates whether that content is answerable from `wiki/` alone. In Codex, the default compile behavior is `ingest -> post-compile coverage-review` unless the user explicitly asks for `compile-only`.

When the user adds a new source to `raw/` or asks for ingest:

1. Read `raw/raw-index.md` to see which sources are not yet tracked or finished.
2. Read the target raw source.
3. Decide one primary `cluster`. If the page clearly belongs to one stable topic line inside that cluster, also assign one `subcluster`.
4. Clarify the intended summary angle with the user when needed.
5. Create a source summary under `wiki/sources/`.
6. Create or update related concept and entity pages.
7. Add high-confidence cross-references with explicit relation notes.
8. Update the relevant page under `wiki/subclusters/` when the page belongs to an established subcluster.
9. Update the relevant page under `wiki/clusters/`.
10. Update `wiki/overview.md` only if the top-level cluster entrance layer itself changes.
11. Append the ingest event to `wiki/log.md`.
12. Mark the source as `done` in `raw/raw-index.md`, record the source page, and leave `驗證日期` empty until validation passes.
13. If running in Codex and the user did not explicitly ask for `compile-only`:
    - For a single source, spawn a fresh coverage-review agent after ingest completes.
    - For a batch, finish ingest for all selected sources first, then review each source in source order.
    - Pass only minimal handoff metadata: `raw_file`, `source_page`, `cluster`, optional `subcluster`, `touched_pages`, `compiled_at`, optional `batch_id`, and `navigation_entry=wiki/overview.md`.
    - Treat compile as complete only after the review gate finishes.
    - If review passes, fill `驗證日期`.
    - If review fails after the allowed repair cycles, keep `驗證日期` empty and record the unresolved gap summary in `raw/raw-index.md` notes.

## Query

When the user asks a question:

1. Start from `wiki/overview.md`, then move into the relevant cluster page, and then the relevant subcluster page when one exists.
2. Read the relevant wiki pages before touching `raw/`.
3. Answer from the wiki, citing the relevant pages.
4. Cross clusters only when the answer truly requires it.
5. If the answer has durable value, ask whether it should be archived to `wiki/analyses/`.
6. If archived, update the relevant cluster page and `wiki/log.md`, and update `wiki/overview.md` only if the entrance layer changes.

## Coverage Review

Coverage review remains a separate phase. In Codex post-compile mode, run it in a fresh agent/session that rebuilds context from the repository instead of relying on the ingest conversation. The wiki-only answer pass should enter through `wiki/overview.md` and the relevant cluster page, not a deleted global index.

When the user asks whether a source is covered well enough, or after major edits:

1. Resolve the target using `raw/raw-index.md`, source frontmatter, `wiki/overview.md`, the corresponding cluster page, and the subcluster page when applicable.
2. Generate source-grounded questions that a good wiki should answer without reopening the raw file during answer time.
3. Run a wiki-only answer pass.
4. Grade each answer as `pass`, `partial`, or `fail`.
5. Diagnose the gap type for every `partial` or `fail`.
6. If allowed, repair high-confidence omissions or routing problems.
7. Re-run regression on the same questions after repair.
8. Run holdout evaluation after self-heal when stricter validation is required.
9. Update the affected subcluster pages, cluster pages, `wiki/log.md`, and `raw/raw-index.md` when appropriate.

Default acceptance threshold:

- At least `80%` of primary questions pass
- Average navigation cost stays at `<= 3` pages
- No failed question is central to the source's main contribution

## Capture

When ingest or query work surfaces an open question worth preserving:

1. Create a page under `wiki/questions/`.
2. Link only high-confidence related pages.
3. Add the question to the relevant subcluster page when it improves navigation, and to the cluster page only when the broader entrance also benefits.
4. Update `wiki/log.md`, and update `wiki/overview.md` only if the top-level cluster entrance layer changes.
5. When the question is resolved, mark it accordingly and archive the answer under `wiki/analyses/` if appropriate.
