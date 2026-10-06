# Workflows

This file fixes the harness-agnostic **phase contract**: which phases exist, who owns them, what gates completion, and what bookkeeping each phase requires. Execution detail lives in the skills (`.claude/skills/`). When this file and a skill disagree on execution mechanics, the skill wins. When they disagree on the phase contract, this file wins.

Every workflow runs inside one domain at a time. Paths below use `<domain>/` for the domain folder (`repository-structure.md` → Identifying Domains).

## Ingest (`compile` skill, wrapped by `sweep` for discovery)

1. **Resolve targets.** Register untracked files in each domain's `<domain>/raw/raw-index.md` as `pending`. The domain of a source is the folder that holds it. In interactive mode, confirm the selection with the user.
2. **Analyst phase.** Read-only analysts each read the raw source and write a findings note to `.claude/scratch/findings/`. The team comes from `<domain>/GUIDE.md` → Analysts. When that section is absent, the default team runs:
   - `theory-context-analyst` — theory and method context
   - `derivation-checker` — derivation audit
   - `experiment-synthesizer` — experiment and results synthesis
3. **Write phase.** `compile-runner` turns the findings into wiki pages under `<domain>/wiki/` (compile skill W1–W5). It sets the source's raw-index row to `done` with `Ingested On` and `Source Page` filled, and leaves `Validated On` empty.
4. **Review phase.** Coverage review runs per source. Compile is complete only after the review finishes.
   - On pass, the reviewer fills `Validated On`.
   - On fail after the repair-cycle cap, `Validated On` stays empty and `Notes` gets a short unresolved-gap phrase.

For a batch, finish ingest for every selected source first, then review each source in source order.

Phase handoffs carry metadata only (compile skill Step 4), never raw-source excerpts or ingest reasoning.

## Coverage Review (`coverage-review` skill)

Coverage review is a blind-retrieval evaluation. It runs in fresh contexts that rebuild state from the repository, never from the ingest conversation. It has three roles:

- **Reviewer (referee).** Reads the raw source, writes the questions and a private answer key, grades blind answers, self-heals high-confidence gaps, and issues the `Ready` verdict and the bookkeeping.
- **Blind answerer.** A fresh agent for every pass (initial, regression, holdout). It receives only the question file and answers from `<domain>/wiki/` alone, entering through `<domain>/wiki/overview.md`.
- **Orchestrator.** Relays file paths between the two roles and enforces the repair-cycle cap.

The answer pass lives in a separate context because an agent that has read the raw source cannot un-know it.

Acceptance threshold:

- At least `80%` of primary questions pass
- Average navigation cost stays at `<= 3` pages
- No failed question is central to the source's main contribution

Repairs update the affected subcluster pages, `<domain>/wiki/overview.md` when the entrance layer changes, `<domain>/wiki/log.md`, and `<domain>/raw/raw-index.md`.

## Query (`query` skill)

1. Navigate `index.md` → `<domain>/wiki/overview.md` → subcluster page (when one exists) → leaf pages. Read the wiki before touching `<domain>/raw/`.
2. Answer from the wiki and cite the pages. Read more than one domain only when the answer requires it. Pages in different domains never gain links to each other.
3. When the answer has durable value, offer to archive it in `<domain>/wiki/analyses/` of the domain it mainly draws on. On archive, update the relevant subcluster page or `<domain>/wiki/overview.md` and `<domain>/wiki/log.md`.

## Capture

When ingest or query work surfaces an open question worth preserving:

1. Create a page under `<domain>/wiki/questions/` and link only high-confidence related pages in the same domain.
2. Add the question to the relevant subcluster page when that improves navigation, and to `<domain>/wiki/overview.md` only when the broader entrance benefits too.
3. Update `<domain>/wiki/log.md`.
4. When the question is resolved, mark the page resolved and archive the answer under `<domain>/wiki/analyses/` when it has durable value.
