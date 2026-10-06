---
name: query
description: Query Wiki — answer user questions by researching wiki pages and optionally archive valuable synthesized answers. Use when the user asks a question about knowledge stored in the wiki.
---

# Query Wiki

Answer user questions by researching the wiki, and archive valuable answers.

## When to Use

- User asks a question about a topic that may be covered in the wiki
- User wants to find or synthesize information across multiple wiki pages
- User asks what the wiki says about a concept, entity, or source

## Input

Use the user's request text as the query topic.

## Workflow

### Step 1: Locate relevant pages
- Read `index.md` at the repository root to choose the relevant domain. `<domain>` below stands for its key
- Read `<domain>/wiki/overview.md`, then the relevant `<domain>/wiki/subclusters/` page when one exists, before diving into leaf pages
- Select the most relevant wiki pages based on query keywords and entrance routing (not raw sources)
- If entrance routing is insufficient, use Grep to search `<domain>/wiki/`
- When the question spans domains, repeat this step for each domain it needs. Read across domains freely, but never add links between pages of different domains

### Step 2: Research and synthesize
- Read all relevant wiki pages
- Only fall back to `<domain>/raw/` source files if wiki pages reference them and they help answer the question
- Synthesize information from multiple pages into a structured answer
- Cite sources as clickable Markdown links with repo-relative paths, e.g. `見 [page-name](deep-learning-research/wiki/concepts/page-name.md)` — `[[wiki-links]]` are for wiki content files only, not chat replies (see `rules/writing-style.md`)

### Step 3: Answer the user
- Conclusion first, then details
- Cite the wiki pages that informed the answer
- Explain before you cite — a page link must supplement an explanation, never replace it. When the answer involves a concept, module, or entity the user may not already know, give a one-clause plain-language gloss of what it is / what it does at first mention, then attach the link for going deeper. Bad: 「這裡把 module A、B 結合，請見 [page-name](deep-learning-research/wiki/concepts/page-name.md)」without saying what A and B are — the link is for depth, not a substitute for understanding the answer.
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

If worth archiving, ask the user: "This answer may be worth archiving to <domain>/wiki/analyses/ — shall I?" Choose the domain the answer mainly draws on.

### Step 5: Archive (after user approval)
- Create an analysis page in `<domain>/wiki/analyses/` of the chosen domain
- Filename: `analysis-{kebab-case-topic}.md`
- Must include YAML frontmatter:
  ```yaml
  ---
  title: Analysis title
  type: analysis
  created: {today}
  updated: {today}
  sources: [raw files referenced by cited wiki pages, as raw/{filename}]
  subcluster: {subcluster-key} # optional; a key from <domain>/GUIDE.md → Subcluster Keys
  tags: [relevant, tags]
  ---
  ```
- Add cross-references `[[analysis-name]]` in related wiki pages of the same domain only
- Update the relevant `<domain>/wiki/subclusters/` page when the archived analysis improves navigation there
- Update `<domain>/wiki/overview.md` only if the entrance layer itself changes
- Append to `<domain>/wiki/log.md`:
  ```
  ## [YYYY-MM-DD] query | Question summary
  - Archived page: [[analysis-name]]
  - Referenced pages: [[page1]], [[page2]]
  - Summary: one-line summary of the analysis
  ```

## Rules

- Prefer answering from wiki pages — do not jump straight to raw sources
- If new questions worth exploring are discovered, propose creating a `<domain>/wiki/questions/` page
