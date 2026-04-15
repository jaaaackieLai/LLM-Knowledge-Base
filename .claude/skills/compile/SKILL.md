---
name: compile
description: Compile Raw Source into Wiki. Use when the user wants to ingest a raw source file into the wiki, or when new files are found under raw/ that haven't been processed yet.
---

# Compile Raw Source into Wiki

Ingest raw sources into the wiki, then gate completion on an independent coverage review unless the user explicitly asks for `compile-only`.

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

## Workflow

Complete ingest for each selected source first. If more than one source is selected, treat them as a batch and run post-compile review only after the whole batch finishes ingest.

### Step 0: Auto-discover new files (only when no file is specified)

- Scan all files under `raw/` (excluding `raw-index.md` and `assets/`)
- Read `raw/raw-index.md` for the list of already-tracked files
- Compare to find files not yet recorded in `raw/raw-index.md`
- Add new files to `raw/raw-index.md` with `pending` status
- List all `pending` files and ask the user whether to ingest all or select specific ones

### Step 1: Read and understand

- Read the full source document
- Identify: topic, key concepts, mentioned entities (people/tools/organizations), core arguments

### Step 2: Create source summary page

- Create a summary page in `wiki/sources/`
- Filename: `source-{kebab-case-title}.md`
- Must include YAML frontmatter:

  ```yaml
  ---
  title: Source title
  type: source
  created: {today}
  updated: {today}
  sources: [raw/{filename}]
  cluster: {cluster-key}
  bridges: [] # optional
  tags: [relevant, tags]
  ---
  ```

- Content structure: summary (2-3 paragraphs) -> key takeaways -> connections to other wiki pages with concrete rationale

### Step 3: Update or create concept / entity pages

- For important concepts mentioned in the source:
  - If page already exists in `wiki/concepts/` -> update with new information and references
  - If not -> create new page
- For important entities (people, tools, organizations) mentioned:
  - If page already exists in `wiki/entities/` -> update
  - If not -> create new page
- All pages must have frontmatter and use `[[wiki-link]]` cross-references
- Add a link only when you can justify it in one concrete sentence
- Treat source-explicit, dependency, comparison, author/work, and question/source links as `direct` relations
- Treat editor synthesis as `extended` relations only when the bridge is already supported somewhere in `wiki/`
- If a relation is plausible but not yet supportable, capture it as a question instead of forcing a cross-reference

### Step 4: Capture open questions

- Identify unanswered questions or directions worth exploring
- Create question pages in `wiki/questions/` if warranted

### Step 5: Update cluster entrances, raw index, and log

- Update the relevant page under `wiki/clusters/`
- Update `wiki/overview.md` only if the cluster entrance layer itself changes
- Append to `wiki/log.md`:

  ```
  ## [YYYY-MM-DD] ingest | Source title
  - New pages: `page1`, `page2`
  - Updated pages: `page3`
  - Summary: one-line summary of the source's core content
  ```
- Update `raw/raw-index.md`: set file status to `done`, fill in `Ingested On` and `Source Page`, and leave `Validated On` blank until review passes

### Step 6: Build post-compile review handoff

After ingest for one source, collect the following payload:

```yaml
raw_file: raw/{filename}
source_page: wiki/sources/source-{kebab-case-title}.md
cluster: {cluster-key}
touched_pages:
  - wiki/sources/source-{kebab-case-title}.md
  - wiki/concepts/example-concept.md
compiled_at: {today}
batch_id: compile-{timestamp} # optional
navigation_entry: wiki/overview.md
```

- `touched_pages` must include all newly created or updated wiki pages from this ingest
- Use repo-relative paths in the payload
- Do not include raw-source excerpts or reasoning from the ingest pass

### Step 7: Independent coverage review (spawn subagent)

After compile finishes, **a separate subagent must be spawned to run coverage-review**, to prevent the same instance that just wrote the wiki from grading itself (player-as-referee).

- If `post_review=skip`, stop after Step 5 and report that validation was intentionally skipped
- Use the Agent tool with `subagent_type: coverage-reviewer` (defined in `.claude/agents/coverage-reviewer.md`, dedicated to independent review)
- Single source: spawn one fresh coverage-reviewer subagent after Step 6
- Batch compile: finish ingest for the entire batch first, then spawn one fresh coverage-reviewer subagent per source in source order
- The prompt passed in must include the full handoff payload and whether review-only is required (default to allowing self-heal unless the user explicitly says otherwise)
- The handoff supplies metadata only — no raw excerpts or reasoning from the ingest pass
- Once the subagent returns its report, the main Claude must relay the Report section **verbatim** to the user; do not regrade or rewrite the verdict
- Treat compile as incomplete until each required review finishes
- The review phase is responsible for filling `Validated On` on success or leaving it blank with an unresolved-gap note in `Notes` on failure

Prompt template (fill in actual paths):

```
Run an independent coverage review on the source just ingested:

- raw file: {raw/...}
- source page: {wiki/sources/source-...md}
- cluster: {cluster-key}
- touched_pages: {...}
- compiled_at: {YYYY-MM-DD}
- batch_id: {optional}
- navigation_entry: wiki/overview.md

Follow your agent spec (fully comply with `.claude/skills/coverage-review/SKILL.md`).
Return the final Report section (Score / Primary / Holdout / Actions / Verdict) in the configured wiki language.
```

## Rules

- Never modify files in `raw/` (except `raw/raw-index.md`)
- Every new page must have at least one inbound link from another page
- Summaries: conclusion first, then details -- keep precise and concise
- In related-page sections, use the direct / extended relation labels required by repo rules (see `rules/content-rules.md`)
- Do not use vague notes such as `related`, `see also`, or `same theme` without naming the actual bridge
- Use the relevant cluster page and `wiki/overview.md` instead of any deleted global index
