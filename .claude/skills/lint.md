---
name: lint
description: Lint Wiki — run health checks (orphans, broken links, missing pages, contradictions, frontmatter, relation quality, etc.) and fix issues on request. Use when the user asks to check wiki health or run a specific lint check.
---

# Lint Wiki

Run health checks on the wiki, report issues, and fix them on request.

## When to Use

- User asks to lint, audit, or health-check the wiki
- User asks about orphan pages, broken links, missing cross-references, or contradictions
- User runs `/lint` with or without a specific check name
- After large edits or restructuring that may have introduced inconsistencies

## Input

$ARGUMENTS

- If a specific check is named (e.g. `orphans`, `contradictions`), run only that check
- If nothing is specified, run all checks

## Workflow

### Step 1: Scan wiki overview

- Read `wiki/wiki-index.md` to get the full page list
- Scan all .md files under `wiki/`
- Compare index vs actual files for inconsistencies

### Step 2: Run checks

Execute the following checks in order, reporting results after each:

#### 2a: Orphan pages (orphans)

- Scan all `[[wiki-link]]` references across wiki pages
- Find pages with zero inbound links (excluding wiki-index.md, log.md, overview.md)
- Report: list orphan pages

#### 2b: Broken links (broken-links)

- Scan all `[[wiki-link]]` references
- Find links pointing to non-existent pages
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

#### 2g: Index integrity (index)

- Confirm all wiki pages are listed in `wiki/wiki-index.md`
- Confirm index descriptions match actual page content
- Confirm `raw/raw-index.md` status matches actual ingest state

#### 2h: Frontmatter integrity (frontmatter)

- Confirm all wiki pages have YAML frontmatter
- Check required fields: title, type, created, updated, sources, tags

#### 2i: Relation quality (relation-quality)

- Scan `## Related Pages`, `## Related Concepts`, and similar sections
- Flag vague relation notes such as `related`, `further reading`, `see also`, or notes that could apply to many pages equally
- Flag grouped links whose single note does not clearly cover each linked page
- Flag notes that overstate certainty when only loose thematic overlap is evident
- Report: suspicious weak or false links for human review

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

- Append to `wiki/log.md`:

  ```
  ## [YYYY-MM-DD] lint | Health check
  - Checks run: N
  - Issues found: N
  - Issues fixed: N
  - Summary: one-line summary of this lint pass
  ```

## Rules

- Only report issues with high confidence -- avoid false positives
- Fixes are limited to structural issues (links, frontmatter, index). Do not auto-fix content issues
- Contradictions and stale claims are reported only -- they require user judgment
- Relation-quality findings are report-first; only auto-fix when the correction is structurally obvious
