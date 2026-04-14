---
name: compile
description: Compile Raw Source into Wiki. Use when the user wants to ingest a raw source file into the wiki, or when new files are found under raw/ that haven't been processed yet.
---

# Compile Raw Source into Wiki

Ingest raw sources into the wiki.

## When to Use

- User asks to ingest, compile, or process a raw source file
- User drops a new file into `raw/` and wants it added to the wiki
- User runs `/compile` with or without a file argument
- New files are discovered under `raw/` that are not yet in `raw/raw-index.md`

## Input

$ARGUMENTS

- If a file path is specified (e.g. `raw/my-article.md`), process only that file
- If no file is specified, run Step 0 to auto-discover new files, then process each one

## Workflow

For each source to be ingested, execute the following steps in order:

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

### Step 5: Update index and log

- Update `wiki/wiki-index.md`: add all new/updated pages under their respective categories
- Update `wiki/overview.md`: reflect newly added knowledge and statistics
- Append to `wiki/log.md`:

  ```
  ## [YYYY-MM-DD] ingest | Source title
  - New pages: [[page1]], [[page2]]
  - Updated pages: [[page3]]
  - Summary: one-line summary of the source's core content
  ```
- Update `raw/raw-index.md`: set file status to `done`, fill in ingest date and wiki page link

## Rules

- Never modify files in `raw/` (except `raw/raw-index.md`)
- Every new page must have at least one inbound link from another page
- Summaries: conclusion first, then details -- keep precise and concise
- In related-page sections, prefer relation notes such as `Direct relation: ...` or `Extended relation: ...`
- Do not use vague notes such as `related`, `see also`, or `same theme` without naming the actual bridge
