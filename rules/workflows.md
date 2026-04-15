# Workflows

This file defines the operational workflows for ingest, query, coverage review, and question capture.

## Ingest

In this repository, `compile` and `ingest` refer to the same workflow.

When the user adds a new source to `raw/` or asks for ingest / compile:

1. Read `raw/raw-index.md` to see which sources are not yet tracked or finished.
2. Read the target raw source.
3. Decide one primary `cluster`, and add `bridges` only when clearly justified.
4. Clarify the intended summary angle with the user when needed.
5. Create a source summary under `wiki/sources/`.
6. Create or update related concept and entity pages.
7. Add high-confidence cross-references with explicit relation notes.
8. Update the relevant page under `wiki/clusters/`.
9. Update `wiki/overview.md` only if the cluster entrance layer itself changes.
10. Append the ingest event to `wiki/log.md`.
11. Mark the source as `done` in `raw/raw-index.md`.
12. Unless the user explicitly asks for `compile-only`, `just ingest`, `ingest without review`, or equivalent, ingest must continue into post-ingest coverage review.
13. Post-ingest coverage review must be run by a fresh independent `coverage-reviewer` agent or subagent in the active environment, not by the same agent instance that performed the ingest.
14. The reviewer must rebuild context from repository state using only minimal routing handoff data. It must not rely on ingest-session memory or reuse raw-source reasoning from the ingest pass.
15. For a single source, run one independent review after ingest. For a batch ingest, finish ingest for the whole batch first, then run one independent review per source in source order.
16. Ingest / compile is not complete until every required independent review finishes.
17. The post-ingest review reuses the normal coverage-review workflow and threshold. Do not create a separate review standard for ingest / compile.
18. On review success, fill `Validated On` in `raw/raw-index.md`. On review failure, leave `Validated On` blank and record a concise unresolved-gap note in `Notes`.

## Query

When the user asks a question:

1. Start from `wiki/overview.md`, then move into the relevant cluster page.
2. Read the relevant wiki pages before touching `raw/`.
3. Answer from the wiki, citing the relevant pages.
4. Cross clusters only when the answer truly requires it.
5. If the answer has durable value, ask whether it should be archived to `wiki/analyses/`.
6. If archived, update the relevant cluster page and `wiki/log.md`, and update `wiki/overview.md` only if the entrance layer changes.

## Coverage Review

When the user asks whether a source is covered well enough, or after major edits:

1. Resolve the target using `raw/raw-index.md`, source frontmatter, and the corresponding cluster page.
2. Generate source-grounded questions that a good wiki should answer without reopening the raw file during answer time.
3. Run a wiki-only answer pass.
4. Grade each answer as `pass`, `partial`, or `fail`.
5. Diagnose the gap type for every `partial` or `fail`.
6. If allowed, repair high-confidence omissions or routing problems.
7. Re-run regression on the same questions after repair.
8. Run holdout evaluation after self-heal when stricter validation is required.
9. Update the affected cluster pages, `wiki/log.md`, and `raw/raw-index.md` when appropriate.

Default acceptance threshold:

- At least `80%` of primary questions pass
- Average navigation cost stays at `<= 3` pages
- No failed question is central to the source's main contribution

## Capture

When ingest or query work surfaces an open question worth preserving:

1. Create a page under `wiki/questions/`.
2. Link only high-confidence related pages.
3. Add the question to the relevant cluster page when it improves navigation.
4. Update `wiki/log.md`, and update `wiki/overview.md` only if the cluster entrance layer changes.
5. When the question is resolved, mark it accordingly and archive the answer under `wiki/analyses/` if appropriate.
