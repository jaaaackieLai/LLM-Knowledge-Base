---
name: query
description: Use when the user asks to query the LLM Knowledge Base wiki, answer from wiki pages, synthesize wiki knowledge, or archive a valuable wiki analysis.
---

# Query Wiki

Answer user questions by researching the wiki, and archive valuable answers.

## Input

Use the user's request text as the query topic.

- The user's question or query topic

## Workflow

### Step 1: Locate relevant pages

- Read `wiki/overview.md` to choose the relevant cluster entrance
- Read the relevant `wiki/clusters/` page before diving into leaf pages
- Select the most relevant wiki pages based on query keywords and cluster routing (not raw sources)
- If entrance routing is insufficient, use Grep to search the `wiki/` directory

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

If worth archiving, ask the user: "This answer may be worth archiving to wiki/analyses/ -- shall I?"

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
  cluster: {primary-cluster}
  bridges: [] # optional
  tags: [relevant, tags]
  ---
  ```

- Add cross-references `[[analysis-name]]` in related wiki pages
- Update the relevant `wiki/clusters/` page when the archived analysis improves navigation there
- Update `wiki/overview.md` only if the entrance layer itself changes
- Append to `wiki/log.md`:

  ```markdown
  ## [YYYY-MM-DD] query | Question summary
  - Archived page: `analysis-name`
  - Referenced pages: `page1`, `page2`
  - Summary: one-line summary of the analysis
  ```

## Rules

- Prefer answering from wiki pages -- do not jump straight to raw sources
- If new questions worth exploring are discovered, propose creating a `wiki/questions/` page
