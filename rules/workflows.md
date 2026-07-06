# Workflows

This file defines the operational workflows for ingest, query, coverage review, and question capture.

It is harness-agnostic: it fixes the **phase contract** — which phases exist, who owns them, what gates completion, and what bookkeeping is required. The executable, per-harness detail lives in the skill files (`.claude/skills/` is canonical; `.agents/skills/` mirrors it for Codex). If this file and a skill disagree on execution mechanics, the skill wins; if they disagree on the phase contract, this file wins.

## Ingest (compile)

Treat ingest and coverage review as separate phases. Ingest creates or updates wiki content from `raw/`. Coverage review validates whether that content is answerable from `wiki/` alone, and **gates compile completion**.

Pipeline (full procedure in the `compile` skill; the `sweep` skill wraps it with automatic discovery):

1. **Resolve targets.** Read `raw/raw-index.md` for sources not yet `done`; register untracked files as `pending`. In interactive mode, confirm the file selection and summary angle with the user when needed.
2. **Analyst phase.** Three read-only analysts — theory & method context, derivation audit, experiment & results synthesis — each read the raw source and write a findings note to `.claude/scratch/findings/`. They never edit wiki pages.
3. **Write phase.** The writer (compile-runner) turns the findings into wiki pages: decide one primary `cluster` (and at most one `subcluster`), create the source summary under `wiki/sources/`, create or update concept/entity pages, add high-confidence cross-references with explicit relation notes, capture open questions, update the relevant subcluster and cluster entrance pages, update `wiki/overview.md` only if the entrance layer itself changes, append the ingest event to `wiki/log.md`, and set the source's `raw/raw-index.md` row to `done` with `Ingested On` and `Source Page` filled — leaving `Validated On` empty.
4. **Review phase.** An independent coverage review runs per source in fresh contexts (see Coverage Review below). Treat compile as complete only after the review gate finishes.
   - If review passes, the reviewer fills `Validated On`.
   - If review fails after the allowed repair cycles, `Validated On` stays empty and a concise unresolved-gap summary goes into the `Notes` column.

Batch rule: finish ingest for all selected sources first, then review each source in source order.

Phase handoffs carry metadata only: `raw_file`, `source_page`, `cluster`, optional `subcluster`, `touched_pages`, `compiled_at`, optional `batch_id`, and `navigation_entry=wiki/overview.md` — never raw-source excerpts or reasoning from the ingest pass.

## Query

When the user asks a question:

1. Start from `wiki/overview.md`, then move into the relevant cluster page, and then the relevant subcluster page when one exists.
2. Read the relevant wiki pages before touching `raw/`.
3. Answer from the wiki, citing the relevant pages.
4. Cross clusters only when the answer truly requires it.
5. If the answer has durable value, ask whether it should be archived to `wiki/analyses/`.
6. If archived, update the relevant cluster page and `wiki/log.md`, and update `wiki/overview.md` only if the entrance layer changes.

## Coverage Review

Coverage review is a separate phase run in fresh contexts that rebuild state from the repository — never from the ingest conversation. It is a **blind-retrieval evaluation** with three roles (full procedure in the `coverage-review` skill):

- **Reviewer (referee).** Reads the raw source; generates source-grounded questions plus a private answer key (written to scratch); grades the blind answers against the key and the wiki; diagnoses gap types; self-heals high-confidence gaps when allowed; generates holdout questions after self-heal; issues the final `Ready` verdict and the `raw/raw-index.md` bookkeeping.
- **Blind answerer.** A separate fresh agent that receives **only the question file** — never the answer key, the raw source, or the ingest context. It answers strictly from `wiki/`, entering through `wiki/overview.md`, and records the answer, pages used, and page count per question. It must not read `raw/`, `wiki/log.md`, or any scratch file other than its assigned question and answer files. A fresh blind answerer is used for every pass (initial, regression, holdout).
- **Orchestrator.** The main thread relays question-file and answer-file paths between reviewer and blind answerer, and enforces the repair-cycle cap.

The separation exists because an agent that has read the raw source cannot un-know it: a "wiki-only" answer pass inside the same context is blind in name only.

When the user asks whether a source is covered well enough, or after major edits:

1. Resolve the target using `raw/raw-index.md`, source frontmatter, `wiki/overview.md`, the corresponding cluster page, and the subcluster page when applicable.
2. Reviewer generates source-grounded primary questions that a good wiki should answer without reopening the raw file, and writes the questions (without answers) and the answer key to scratch.
3. A fresh blind answerer runs the wiki-only answer pass on the question file.
4. Reviewer grades each answer as `pass`, `partial`, or `fail`, verifying that cited wiki pages actually support the answers.
5. Reviewer diagnoses the gap type for every `partial` or `fail`.
6. If allowed, reviewer repairs high-confidence omissions or routing problems.
7. After repair, a fresh blind answerer re-runs the same primary questions (regression), and the reviewer re-grades.
8. After a passing regression, a fresh blind answerer runs the holdout question set, and the reviewer grades it for the final verdict.
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
