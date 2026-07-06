---
name: compile
description: Compile Raw Source into Wiki. Use when the user wants to ingest a raw source file into the wiki, or when new files are found under raw/ that haven't been processed yet.
---

# Compile Raw Source into Wiki

Ingest raw sources into the wiki, then gate completion on an independent, blind coverage review unless the user explicitly asks for `compile-only`.

This skill runs on the **main thread** as the orchestrator. The orchestrator spawns every phase subagent and relays scratch-file paths between them; no subagent spawns another. The orchestrator itself does **not** read the full raw source and does **not** write wiki pages — the analysts, the writer, and the reviewer do.

## When to Use

- User asks to ingest, compile, or process a raw source file
- User drops a new file into `raw/` and wants it added to the wiki
- New files are discovered under `raw/` that are not yet in `raw/raw-index.md`

## Input

Use the user's request text to determine which raw source files to ingest and whether post-review should run.

- If a file path is specified (e.g. `raw/my-article.md`), process only that file
- If no file is specified, run Step 0 to auto-discover new files, then process each one
- Default `post_review=auto`
- Set `post_review=skip` only when the user explicitly says `compile-only`, `just ingest`, `ingest without review`, `--no-review`, or equivalent

## Scratch file naming (mandatory)

Derive a deterministic `<slug>` from the raw filename (short kebab-case identifier). Always pass **exact paths** to every subagent — never let an agent invent its own filename, so re-runs overwrite instead of accumulating:

- Findings: `.claude/scratch/findings/<slug>-theory.md`, `<slug>-derivation.md`, `<slug>-experiment.md`
- Coverage: `.claude/scratch/coverage/<slug>-primary-questions.md`, `<slug>-primary-key.md`, `<slug>-primary-answers.md`, `<slug>-regression-answers.md`, `<slug>-holdout-questions.md`, `<slug>-holdout-key.md`, `<slug>-holdout-answers.md`

## Workflow (orchestrator)

Complete ingest for each selected source first. If more than one source is selected, treat them as a batch: finish Steps 1–4 for every source, then run Step 5 per source in source order.

### Step 0: Auto-discover new files (only when no file is specified)

- Scan all files under `raw/` (excluding `raw-index.md` and `assets/`)
- Read `raw/raw-index.md` for the list of already-tracked files
- Compare to find files not yet recorded in `raw/raw-index.md`
- Add new files to `raw/raw-index.md` with `pending` status
- List all `pending` files and ask the user whether to ingest all or select specific ones

### Step 1: Verify targets

- Confirm each target file exists under `raw/` and has a row in `raw/raw-index.md` (register as `pending` if missing)
- Do not read the source content on the main thread

### Step 2: Analyst team (parallel, Opus 4.8)

In a single message, spawn all three analysts, passing the raw source path and the **exact** findings output path each must use (see Scratch file naming):

- **theory-context-analyst** — problem, method lineage, core idea/assumptions, concepts/entities to page
- **derivation-checker** — transcribe & re-derive key equations; flag Error/Assumption/Gap
- **experiment-synthesizer** — setup, headline results, and the ablation read of which component drives the gains

Each analyst writes its findings note to its assigned file and returns only the path + a short gist. Collect the three findings-file paths (do not expect full notes inline).

### Step 3: Write phase (spawn compile-runner)

Spawn one **compile-runner** per source, passing the raw path and the three findings paths. It executes the Write Phase Specification below (source summary → concept/entity pages → open questions → entrances → log → raw index) and returns:

- the coverage-review handoff payload (see Step 4 format)
- a `Claim–Evidence Conflicts` section — relay it to the user

### Step 4: Collect the handoff payload

Per source, the payload from compile-runner must contain:

```yaml
raw_file: raw/{filename}
source_page: wiki/sources/source-{kebab-case-title}.md
cluster: {cluster-key}
subcluster: {subcluster-key} # optional
touched_pages:
  - wiki/sources/source-{kebab-case-title}.md
  - wiki/concepts/example-concept.md
compiled_at: {today}
batch_id: compile-{timestamp} # optional
navigation_entry: wiki/overview.md
```

- `touched_pages` must include all newly created or updated wiki pages from this ingest
- Repo-relative paths only; no raw-source excerpts or reasoning from the ingest pass

### Step 5: Blind coverage review (orchestrated, per source in source order)

If `post_review=skip`, stop after Step 4 and report that validation was intentionally skipped. Otherwise run this protocol; it exists because an agent that has read the raw source cannot un-know it — the answer pass must live in a context that has **never seen the source**.

1. **Spawn coverage-reviewer** with the full handoff payload, the `raw_file` path, and the exact coverage scratch paths. It generates the primary questions, writes the question file (questions only) and the private answer key to scratch, and returns `AWAITING BLIND PASS` + the question-file path.
2. **Spawn a fresh blind-answerer** with only: the question-file path, the answers output path, and `navigation_entry: wiki/overview.md`. Never mention the raw file or the source page in its prompt.
3. **Continue the same coverage-reviewer** (SendMessage) with the answers path. It grades and returns either the final Report (`Ready: yes|no`) or `AWAITING REGRESSION PASS` after self-healing (it will have written the holdout question/key files by then).
4. On `AWAITING REGRESSION PASS`: spawn a fresh blind-answerer on the **same primary question file** (answers to the regression-answers path), then continue the reviewer. If regression passes, it returns `AWAITING HOLDOUT PASS`: spawn a fresh blind-answerer on the holdout question file, then continue the reviewer for the final verdict.
5. Repair-cycle cap: at most two self-heal cycles, enforced by the orchestrator. If agent continuation is unavailable, spawn a fresh coverage-reviewer and point it at the scratch state files to rebuild.
6. Relay the reviewer's final Report section **verbatim** to the user; do not regrade or rewrite the verdict.
7. The reviewer fills `Validated On` in `raw/raw-index.md` on `Ready: yes`, or leaves it blank with an unresolved-gap note in `Notes` on `Ready: no`. Treat compile as incomplete until each required review finishes.

Blind-answerer prompt template (fill in actual paths; include nothing else):

```
Run a wiki-only blind answer pass.

- questions file: .claude/scratch/coverage/{slug}-primary-questions.md
- write your answers to: .claude/scratch/coverage/{slug}-primary-answers.md
- navigation entry: wiki/overview.md

Follow your agent spec. Use only wiki/ content.
```

Coverage-reviewer prompt template:

```
Run an independent blind coverage review on the source just ingested:

- raw file: {raw/...}
- source page: {wiki/sources/source-...md}
- cluster: {cluster-key}
- touched_pages: {...}
- compiled_at: {YYYY-MM-DD}
- batch_id: {optional}
- navigation_entry: wiki/overview.md
- scratch paths: {the seven coverage paths for this slug}

Follow your agent spec (fully comply with `.claude/skills/coverage-review/SKILL.md`).
You are the referee: generate questions and grade blind answers, but never run the
answer pass yourself — return AWAITING BLIND PASS and wait for the orchestrator.
Return the final Report section (Score / Primary / Holdout / Actions / Verdict) in the configured wiki language.
```

## Write Phase Specification (executed by compile-runner)

### W1: Read inputs

- Read the three findings notes first; treat them as the primary analyzed input
- **PDF**: read directly with the Read tool — it renders each page to an image via `pdftoppm` (poppler). Use the Read `pages` parameter for long PDFs. Consult the source for exact quotes/numbers when precision matters
- **Markdown / plain text**: use the raw path directly
- If findings files are not supplied, read and understand the source yourself

### W2: Create source summary page

- Filename: `source-{kebab-case-title}.md` (topic-oriented, no venue/year prefix — content-rules rule 18)
- Must include YAML frontmatter:

  ```yaml
  ---
  title: Source title
  type: source
  created: {today}
  updated: {today}
  sources: [raw/{filename}]
  cluster: {cluster-key}
  subcluster: {subcluster-key} # optional
  bridges: [] # optional
  tags: [relevant, tags]
  ---
  ```

- Content structure: summary (2-3 paragraphs) -> key takeaways -> connections to other wiki pages with concrete rationale

### W3: Update or create concept / entity pages

- For important concepts mentioned in the source:
  - If page already exists in `wiki/concepts/` -> update with new information and references
  - If not -> create new page (only when it clears the promotion threshold in `rules/writing-style.md`)
- For important entities (people, tools, organizations) mentioned:
  - If page already exists in `wiki/entities/` -> update
  - If not -> create new page
- All pages must have frontmatter and use `[[wiki-link]]` cross-references
- Add a link only when you can justify it in one concrete sentence
- Treat source-explicit, dependency, comparison, author/work, and question/source links as `direct` relations
- Treat editor synthesis as `extended` relations only when the bridge is already supported somewhere in `wiki/`
- If a relation is plausible but not yet supportable, capture it as a question instead of forcing a cross-reference

### W4: Capture open questions

- Identify unanswered questions or directions worth exploring
- Create question pages in `wiki/questions/` if warranted

### W5: Update cluster entrances, raw index, and log

- Update the relevant page under `wiki/subclusters/` when the page belongs to an established subcluster, and the relevant page under `wiki/clusters/`
- Update `wiki/overview.md` only if the cluster entrance layer itself changes
- Append to `wiki/log.md`:

  ```
  ## [YYYY-MM-DD] ingest | Source title
  - New pages: `page1`, `page2`
  - Updated pages: `page3`
  - Summary: one-line summary of the source's core content
  ```
- Update `raw/raw-index.md`: set file status to `done`, fill in `Ingested On` and `Source Page`, and leave `Validated On` blank until review passes. Keep the Notes cell to one short phrase (content-rules rule 20)

## Rules

- Never modify files in `raw/` (except `raw/raw-index.md`)
- Every new page must have at least one inbound link from another page
- Summaries: conclusion first, then details -- keep precise and concise
- No naked jargon: enforce the first-appearance-term policy in `rules/writing-style.md`
- In related-page sections, use the direct / extended relation labels required by repo rules (see `rules/content-rules.md`)
- Do not use vague notes such as `related`, `see also`, or `same theme` without naming the actual bridge
- Use the relevant cluster page and `wiki/overview.md` instead of any deleted global index
- **Scratch lifecycle.** Analysts write findings under `.claude/scratch/findings/`; the review protocol writes question/key/answer files under `.claude/scratch/coverage/`. These stay in-repo through the write and review phases. **Never move or delete scratch files** — the user's no-deletion policy blocks it. After a passing review, list the stale scratch files for the user to clean manually. Filenames are deterministic per source (orchestrator always passes exact paths), so re-runs overwrite rather than accumulate.
