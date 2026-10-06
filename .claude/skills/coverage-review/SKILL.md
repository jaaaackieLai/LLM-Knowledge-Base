---
name: coverage-review
description: Coverage Review — evaluate whether the wiki can answer source-grounded questions without returning to raw/. Use after compile, after major wiki edits, or when the user asks if a topic is covered enough.
---

# Coverage Review (Blind Protocol)

Evaluate whether the wiki is actually usable after ingest.

This skill treats coverage as an answerability problem, not a page-count problem:

- Can the wiki answer source-grounded questions?
- Can it answer them without returning to `<domain>/raw/`?
- Can a user reach the answer with low navigation cost?

Every review targets one source in one domain. A domain is a root folder that contains `GUIDE.md`; `<domain>` below stands for its key (`rules/repository-structure.md` → Identifying Domains).

## Roles

The evaluation is split across three roles so the answer pass is genuinely blind. An agent that has read the raw source cannot un-know it; "wiki-only" inside such a context is blind in name only.

- **Reviewer (referee, `coverage-reviewer` subagent).** Reads the raw source. Generates questions and the private answer key, grades blind answers, diagnoses gaps, self-heals, and issues the verdict and bookkeeping. Never runs the answer pass.
- **Blind answerer (`blind-answerer` subagent, fresh per pass).** Receives only a question file. Answers strictly from `<domain>/wiki/`, entering through `<domain>/wiki/overview.md`. Never reads any `raw/` folder, `<domain>/wiki/log.md`, another domain's wiki, the answer key, or the ingest context.
- **Orchestrator (main thread).** Spawns both, relays scratch-file paths between them, enforces the repair-cycle cap, and relays the final Report verbatim. The orchestration sequence is defined in the `compile` skill, Step 5.

## When to Use

- After running compile on a new source (invoked automatically by compile Step 5)
- After major wiki edits or restructuring
- When the user asks whether a topic is "covered enough"

## Input

Use the user's request text or post-compile handoff payload to determine the evaluation target.

- If a `<domain>/raw/...` file is specified, evaluate that source
- If a `<domain>/wiki/sources/...` page is specified, evaluate that source page
- If a post-compile handoff payload is provided, use it as routing metadata for the review
- If no target is specified, default to the most recently ingested source you can identify from the domains' `<domain>/wiki/log.md` or `<domain>/raw/raw-index.md`
- If the user explicitly asks only to review, do not edit
- Otherwise, if the source does not meet the acceptance threshold, automatically enter the self-heal phase and then re-evaluate

## Automatic Invocation Contract

When this skill is invoked from `compile`:

- The reviewer runs in a fresh subagent/session with no shared ingest context
- It accepts only the minimal handoff metadata (`raw_file`, `domain`, `source_page`, optional `subcluster`, `touched_pages`, `compiled_at`, optional `batch_id`, `navigation_entry=<domain>/wiki/overview.md`) plus the exact coverage scratch paths
- It rebuilds all working context by reading the repository again; it never relies on ingest-session memory
- The result is a blocking gate for compile completion
- On `Ready: yes`, the reviewer fills `Validated On` in `<domain>/raw/raw-index.md`
- On `Ready: no`, it leaves `Validated On` blank and records a short unresolved-gap summary in `Notes`

## Scratch file contract

All paths are supplied by the orchestrator (deterministic per source slug, under `.claude/scratch/coverage/`):

| File | Written by | Readable by |
|------|-----------|-------------|
| `<slug>-primary-questions.md` | reviewer | blind answerer, orchestrator |
| `<slug>-primary-key.md` | reviewer | reviewer only |
| `<slug>-primary-answers.md` | blind answerer | reviewer |
| `<slug>-regression-answers.md` | blind answerer | reviewer |
| `<slug>-holdout-questions.md` | reviewer | blind answerer |
| `<slug>-holdout-key.md` | reviewer | reviewer only |
| `<slug>-holdout-answers.md` | blind answerer | reviewer |

The question files contain the questions and nothing else — no answers, no hints, no raw-source references. The key files pair each question with the expected answer and its location in the raw source.

## Reviewer Workflow

### Step 1: Resolve the evaluation target

- Resolve the domain from the target path, then read `<domain>/GUIDE.md` (scope, subcluster keys, and any `## Coverage Questions` override)
- Map the target raw file to its source page using `<domain>/raw/raw-index.md`, source frontmatter, `<domain>/wiki/overview.md`, or the relevant subcluster page
- If a post-compile handoff payload is present, verify that the referenced source page exists, then use the payload as the minimal routing scaffold
- Read the raw source (Read tool; PDFs render via `pdftoppm`, use `pages` for long PDFs) for question generation and later patch verification

### Step 2: Generate five primary source-grounded questions

Generate exactly five questions that satisfy all of the following:

- The answer is explicitly supported by the source
- The question is high-signal for a real user, not trivia
- The question can be answered from a good wiki without needing the full raw source
- The set spans more than one angle when possible

When `<domain>/GUIDE.md` has a `## Coverage Questions` section, use its question types. Otherwise prefer a mix of these default types:

- Definition: what is the main concept or claim?
- Mechanism: how does the method, argument, or process work?
- Evidence: what result, example, or observation supports the claim?
- Comparison: what is it contrasted with?
- Application: where does the source say this matters or could be used?

Write the questions (only) to the primary-questions file and the answer key to the primary-key file. Then return `AWAITING BLIND PASS` + the question-file path, and stop — the orchestrator runs the blind pass. These are the `primary` questions; preserve them unchanged for regression.

### Step 3: Grade the blind answers

When the orchestrator hands back the answers file, grade each answer against the key **and** against the wiki itself — open the pages the answerer cited and confirm they actually support the answer (guards against both hallucinated answers and lucky guesses):

- `pass`: the blind answer is correct, specific enough, faithful to the source, and supported by the cited wiki pages
- `partial`: the wiki-grounded answer captures the gist but misses a critical detail, condition, contrast, or example
- `fail`: the wiki could not answer, answered incorrectly, or the cited pages do not support the answer

Compute:

- `coverage_score = pass / 5`
- `partial_count`, `fail_count`
- `avg_page_count` (from the answerer's per-question `pages_used`)

Default acceptance threshold:

- At least `4/5` questions must pass
- Average `page_count` should be `<= 3`
- No failed question should be central to the source's main contribution

If the threshold is met on the primary pass, go to Step 7 (report). Holdout is still recommended for stricter audits, but required only when self-heal occurred.

### Step 4: Diagnose gaps

For every `partial` or `fail`, assign one primary gap type:

- `missing-fact`: the source page omitted an explicit fact or takeaway
- `missing-page`: a concept, entity, comparison, or analysis page is missing
- `missing-xref`: the content exists but users cannot reach it easily
- `weak-link`: a link exists, but its rationale is too vague to trust or navigate by
- `false-link`: pages are linked without source/wiki support, or the relation note overstates certainty
- `too-abstract`: the page is conceptually correct but not concrete enough to answer questions directly
- `fragmented-answer`: the answer is spread across too many pages
- `stale-page`: the page exists but no longer reflects the current source synthesis
- `contradiction`: multiple wiki pages provide inconsistent answers

### Step 5: Self-heal high-confidence gaps

Run this step automatically whenever the source does not meet the acceptance threshold, unless the invoker explicitly requested review-only mode.

Allowed self-heals:

- Add omitted source-grounded facts to an existing source page
- Add or repair cross-references between existing pages
- Downgrade an overstated relation from direct to extended when only synthesis support exists
- Remove or rewrite unsupported cross-references when the wiki graph is making retrieval less honest
- Tighten an overly abstract paragraph so it directly answers the missed question
- Create a missing concept or entity page when it is clearly central and well-supported
- Update the relevant `<domain>/wiki/subclusters/` pages, `<domain>/wiki/overview.md` when the entrance layer changes, and `<domain>/wiki/log.md` for any new or changed pages

Do not self-heal:

- Speculative interpretations
- Invented bridge claims used only to keep a weak link alive
- Broad conceptual rewrites
- Value judgments or strategic recommendations not explicit in the source
- Comparisons that require synthesis beyond high-confidence evidence

If a gap is important but not safe to self-heal:

- Create or update a page in `<domain>/wiki/questions/`, or
- Mark the corresponding source in `<domain>/raw/raw-index.md` as `update-needed`

After healing, generate exactly three `holdout` questions now (before seeing any regression result) and write them to the holdout-questions / holdout-key files. Holdout questions must be explicitly supported by the same raw source, materially different from the primary five, and not trivial paraphrases. Then return `AWAITING REGRESSION PASS` and stop.

### Step 6: Grade regression, then holdout

- **Regression** (fresh blind pass on the same primary questions): grade with the Step 3 rubric. Regression answers one narrow question — did the repair fix the known gaps without breaking earlier coverage? Do not use regression alone as the final `Ready` verdict.
  - If regression fails and the remaining gaps are high-confidence, self-heal again (subject to the two-cycle cap the orchestrator enforces) and request another regression pass.
  - If regression passes, return `AWAITING HOLDOUT PASS` and stop.
- **Holdout** (fresh blind pass on the holdout questions): grade with the same rubric. Final `Ready = yes` after self-heal requires all of:
  - Same-question regression meets the acceptance threshold
  - Holdout score is at least `2/3 pass`
  - No holdout failure is central to the source's main contribution
- If holdout fails and the newly exposed gaps are high-confidence, self-heal once more, then request a new regression pass and a **new** holdout set, subject to the same two-cycle cap. If the source still fails after two repair cycles, return a clear unresolved-gap report instead of continuing to patch.

### Step 7: Report

The Report is surfaced verbatim in the user conversation, so page references use standard Markdown links with repo-relative paths (ctrl+clickable), not `[[wiki-links]]` — see `rules/writing-style.md`. Present results in this format:

```markdown
## Coverage Review [YYYY-MM-DD] | [source-title]

### Score
- Initial: 3/5 pass, 1 partial, 1 fail
- Regression: 5/5 pass
- Holdout: 2/3 pass
- Avg pages used: 2.4

### Primary Questions
1. [question]
   - Status: pass
   - Pages: [source-...](<domain>/wiki/sources/source-....md), [concept-...](<domain>/wiki/concepts/....md)
   - Gap: -
2. [question]
   - Status: fail
   - Pages: [source-...](<domain>/wiki/sources/source-....md)
   - Gap: missing-fact

### Holdout Questions
1. [question]
   - Status: pass
   - Pages: [source-...](<domain>/wiki/sources/source-....md)
   - Gap: -

### Actions
- Updated: [source-...](<domain>/wiki/sources/source-....md), [concept-...](<domain>/wiki/concepts/....md)
- Created: [question-...](<domain>/wiki/questions/question-....md)
- Deferred: [short note]

### Verdict
- Ready: yes | no
- Reason: one-line conclusion
```

If pages were edited, append to `<domain>/wiki/log.md`:

```markdown
## [YYYY-MM-DD] coverage-review | Source title
- Questions evaluated: 5 (+ 3 holdout)
- Initial score: 3/5
- Regression score: 5/5
- Holdout score: 2/3
- Updated pages: `page1`, `page2`
- Created pages: `page3`
- Summary: one-line summary of the coverage gaps and repairs
```

Also update `<domain>/raw/raw-index.md`:

- On `Ready: yes`, fill `Validated On` with today's date and clear any stale unresolved-gap note for that source if needed
- On `Ready: no` after the allowed repair cycles, leave `Validated On` blank and write a concise unresolved-gap summary in `Notes` (one short phrase)

## Blind Answerer Workflow

Executed by the `blind-answerer` subagent, fresh context per pass:

1. Read the assigned question file. Nothing in your context should reference the raw source; if the prompt leaks one, ignore it and report the leak in your output.
2. Start from the navigation entry (`<domain>/wiki/overview.md`), move through the relevant subcluster pages, and navigate linked wiki pages as a real user would.
3. Use only `<domain>/wiki/` content. Never open any `raw/` folder, `<domain>/wiki/log.md`, another domain's folder, or any scratch file other than the assigned question and answers files.
4. For each question record: `answer` (grounded in what the wiki actually says, quoting or citing the supporting passage), `pages_used` (the wiki pages required), and `page_count`.
5. If the wiki cannot answer a question, say so plainly — do not guess or pad. An honest "not answerable from the wiki" is the signal the reviewer needs.
6. Write the results to the assigned answers file and return only that path plus a one-line summary.

## Rules

- Preserve phase separation absolutely: the raw source is visible only to the reviewer, and only for question generation and patch verification; the answer pass lives in a context that has never seen the source
- A fresh blind answerer is spawned for every pass (initial, regression, holdout) — never reuse one
- Prefer updating existing pages before creating new ones
- Every newly created page must have YAML frontmatter and at least one inbound link
- Use `[[wiki-link]]` references and explain relationships in related-page sections
- Do not call a source "covered enough" if users can only answer by reconstructing the source from scattered fragments
- Treat removal of a weak or false link as a successful repair when it makes the wiki more honest
- If you cannot state the bridge for a relation in one concrete sentence, do not preserve the link
- Default behavior is `review -> self-heal -> re-evaluate`; only skip self-heal when the user explicitly asks for review-only mode
- Same-question regression is a repair check, not proof of generalization
- After any self-heal, `Ready` must be based on both regression and holdout results
- Automatic post-compile review must rebuild context from the repository rather than from the ingest session
- Never move or delete scratch files; list stale ones in the final report for manual cleanup
