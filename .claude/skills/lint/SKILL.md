---
name: lint
description: Lint Wiki — run health checks (orphans, broken links, missing pages, contradictions, frontmatter, relation quality, etc.) and fix issues on request. Use when the user asks to check wiki health or run a specific lint check.
---

# Lint Wiki

Run health checks on the wiki, report issues, and fix them on request.

## When to Use

- User asks to lint, audit, or health-check the wiki
- User asks about orphan pages, broken links, missing cross-references, or contradictions
- After large edits or restructuring that may have introduced inconsistencies

## Input

Use the user's request text to decide which check(s) to run.

- If a specific check is named (e.g. `orphans`, `contradictions`), run only that check
- If nothing is specified, run all checks

## Workflow

### Step 1: Scan domains and entrances

- Find every domain with the glob `*/GUIDE.md`. A domain is a root folder that contains `GUIDE.md`; `<domain>` below stands for its key (`rules/repository-structure.md` → Identifying Domains)
- Read `index.md`, then for each domain read `<domain>/GUIDE.md`, `<domain>/wiki/overview.md`, and all pages under `<domain>/wiki/subclusters/` to get the entrance structure
- Scan all .md files under each `<domain>/wiki/`
- Compare entrance pages vs actual files for inconsistencies

Run checks 2a–2j per domain, and report each finding with its domain. Check 2k compares domains with each other.

### Step 2: Run checks

Execute the following checks in order, reporting results after each:

#### 2a: Orphan pages (orphans)

- Scan all `[[wiki-link]]` references across wiki pages
- Find pages with zero inbound links from the same domain (excluding `log.md` and `overview.md`)
- Report: list orphan pages

#### 2b: Broken links (broken-links)

- Scan all `[[wiki-link]]` references
- Find links pointing to pages that do not exist in the same domain
- Report: list broken links and the pages containing them

#### 2c: Missing pages (missing-pages)

- Scan all page content for important concepts or entities mentioned multiple times but lacking their own page
- Report: list suggested pages to create

#### 2d: Missing cross-references (missing-xrefs)

- Find pages with related content that are not linked to each other
- Report: list suggested links to add

#### 2e: Contradictions (contradictions)

- Compare descriptions of the same concept across different pages
- Find inconsistent or contradictory claims
- Report: list contradictions and the pages involved

#### 2f: Stale claims (stale)

- Check whether earlier page claims have been superseded by newer sources
- Compare frontmatter `updated` dates and `sources`
- Report: list potentially outdated content

#### 2g: Entrance integrity (routing | index)

- Confirm `index.md` lists every domain exactly once with a path link to `<domain>/wiki/overview.md`, and lists no folder that lacks `GUIDE.md`
- Confirm each domain has `<domain>/wiki/overview.md`, `<domain>/wiki/log.md`, and `<domain>/raw/raw-index.md`
- Confirm `<domain>/wiki/overview.md` links to every page under `<domain>/wiki/subclusters/`
- Confirm every `<domain>/wiki/subclusters/` page matches a key in `<domain>/GUIDE.md` → Subcluster Keys
- Confirm every `subcluster` value in frontmatter is a key in `<domain>/GUIDE.md` → Subcluster Keys
- Confirm `<domain>/raw/raw-index.md` status matches actual ingest state
- **Raw-index table integrity**: no filename appears in more than one row; every row has exactly six cells (literal `|` inside a cell must be escaped as `\|`); every row's file actually exists under `<domain>/raw/`; every file under `<domain>/raw/` (excluding `raw-index.md` and `assets/`) has at most one row; status is one of the allowed values

#### 2h: Frontmatter integrity (frontmatter)

- Confirm all wiki pages have YAML frontmatter
- Check required fields: title, type, created, updated, sources, tags
- Confirm `type` is one of the values in `rules/page-formats.md` → Frontmatter Format

#### 2i: Relation quality (relation-quality)

- Scan `## Related Pages`, `## Related Concepts`, and similar sections
- Flag vague relation notes such as `related`, `further reading`, `see also`, or notes that could apply to many pages equally
- Flag grouped links whose single note does not clearly cover each linked page
- Flag notes that overstate certainty when only loose thematic overlap is evident
- Report: suspicious weak or false links for human review

#### 2j: Subcluster promotion threshold (promotion)

- For each `<domain>/wiki/overview.md`, scan its supplementary research-line section (e.g. `## 補充研究線`) and, for each line, count the strongly related pages (the line's source pages plus the concept/question pages they introduce)
- Flag any line whose page count meets the subcluster threshold in `rules/content-rules.md` (3+ strongly related pages) as due for promotion to a formal subcluster
- Flag any domain overview whose supplementary section has grown beyond one link per research line, or whose total link list makes it read like a content page instead of an entrance
- Report: promotion suggestions and entrance-bloat findings for user confirmation (promotion itself is a content change — never auto-fix)

#### 2k: Domain boundaries (domains)

- Flag every `[[link]]` inside a domain that resolves only to a page in another domain. Links stay within one domain (`rules/content-rules.md` → Links)
- Warn when two domains hold wiki pages with the same filename, because Obsidian may resolve a short `[[page-name]]` to the wrong domain
- Report: cross-domain links and duplicate filenames, each with both paths

### Step 3: Summary report

Present all findings in checklist format:

```
## Lint Report [YYYY-MM-DD]

### Passed
- [check name]

### Issues found
- [check name]: N issues
  - Issue 1 description
  - Issue 2 description
```

### Step 4: Fix

- Ask the user: "Auto-fix these issues, or just keep the report?"
- After user approval, fix issues one by one
- Verify wiki consistency after each fix

### Step 5: Update log

- Append to the `<domain>/wiki/log.md` of each domain where a fix was applied:

  ```
  ## [YYYY-MM-DD] lint | Health check
  - Checks run: N
  - Issues found: N
  - Issues fixed: N
  - Summary: one-line summary of this lint pass
  ```

## Rules

- Only report issues with high confidence -- avoid false positives
- Fixes are limited to structural issues (links, frontmatter, entrances). Do not auto-fix content issues
- Contradictions and stale claims are reported only -- they require user judgment
- Relation-quality findings are report-first; only auto-fix when the correction is structurally obvious
