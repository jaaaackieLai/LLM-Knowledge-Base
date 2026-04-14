---
name: query
description: Query Wiki — answer user questions by researching wiki pages and optionally archive valuable synthesized answers. Use when the user asks a question about knowledge stored in the wiki.
---

# Query Wiki

Answer user questions by researching the wiki, and archive valuable answers.

## When to Use

- User asks a question about a topic that may be covered in the wiki
- User wants to find or synthesize information across multiple wiki pages
- User runs `/query` followed by their question
- User asks what the wiki says about a concept, entity, or source

## Input

$ARGUMENTS

- The user's question or query topic

## Workflow

### Step 1: Locate relevant pages
- Read `wiki/wiki-index.md` to scan all page entries
- Select the most relevant wiki pages based on query keywords (not raw sources)
- If index info is insufficient, use Grep to search the `wiki/` directory

### Step 2: Research and synthesize
- Read all relevant wiki pages
- Only fall back to `raw/` source files if wiki pages reference them and they help answer the question
- Synthesize information from multiple pages into a structured answer
- Cite sources using `see [[page-name]]` format

### Step 3: Answer the user
- Conclusion first, then details
- Cite the wiki pages that informed the answer
- If the wiki lacks sufficient information, state the knowledge gap clearly

### Step 4: Archive decision
After answering, evaluate whether the answer is worth archiving:

**Worth archiving:**
- Analysis that synthesized information from multiple pages
- New comparisons or insights produced
- Answers to questions likely to be asked again
- Connections between pages not previously documented

**Not worth archiving:**
- Simple fact lookups (answer already exists on a single page)
- Overly specific or one-off questions

If worth archiving, ask the user: "This answer may be worth archiving to wiki/analyses/ — shall I?"

### Step 5: Archive (after user approval)
- Create an analysis page in `wiki/analyses/`
- Filename: `analysis-{kebab-case-topic}.md`
- Must include YAML frontmatter:
  ```yaml
  ---
  title: Analysis title
  type: analysis
  created: {today}
  updated: {today}
  sources: [raw files referenced by cited wiki pages]
  tags: [relevant, tags]
  ---
  ```
- Add cross-references `[[analysis-name]]` in related wiki pages
- Update `wiki/wiki-index.md`: add entry under Analyses
- Append to `wiki/log.md`:
  ```
  ## [YYYY-MM-DD] query | Question summary
  - Archived page: [[analysis-name]]
  - Referenced pages: [[page1]], [[page2]]
  - Summary: one-line summary of the analysis
  ```

## Rules

- Prefer answering from wiki pages — do not jump straight to raw sources
- If new questions worth exploring are discovered, propose creating a `wiki/questions/` page
