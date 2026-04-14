---
name: coverage-review
description: Coverage Review — evaluate whether the wiki can answer source-grounded questions without returning to raw/. Use after compile, after major wiki edits, or when the user asks if a topic is covered enough.
---

# Coverage Review

Evaluate whether the wiki is actually usable after ingest.

This skill treats coverage as an answerability problem, not a page-count problem:

- Can the wiki answer source-grounded questions?
- Can it answer them without returning to `raw/`?
- Can a user reach the answer with low navigation cost?

## When to Use

- After running `/compile` on a new source
- After major wiki edits or restructuring
- When the user asks whether a topic is "covered enough"
- When the user runs `/coverage-review` with or without a target argument

## Input

$ARGUMENTS

- If a `raw/...` file is specified, evaluate that source
- If a `wiki/sources/...` page is specified, evaluate that source page
- If no target is specified, default to the most recently ingested source you can identify from `wiki/log.md` or `raw/raw-index.md`
- If the user explicitly asks only to review, do not edit
- Otherwise, if the source does not meet the acceptance threshold, automatically enter the self-heal phase and then re-evaluate

## Workflow

### Step 1: Resolve the evaluation target

- Map the target raw file to its source page using `raw/raw-index.md`, source frontmatter, or `wiki/wiki-index.md`
- Read the target source page and the directly related concept, entity, comparison, and question pages linked from it
- Read the original raw source only for question generation and later patch verification

### Step 2: Generate five primary source-grounded questions

Generate exactly five questions that satisfy all of the following:

- The answer is explicitly supported by the source
- The question is high-signal for a real user, not trivia
- The question can be answered from a good wiki without needing the full raw source
- The set spans more than one angle when possible

Prefer a mix of question types:

- Definition: what is the main concept or claim?
- Mechanism: how does the method, argument, or process work?
- Evidence: what result, example, or observation supports the claim?
- Comparison: what is it contrasted with?
- Application: where does the source say this matters or could be used?

Keep a private note of where each answer comes from in the raw source. Do not use the raw source during the wiki-only answer phase.
These are the `primary` questions. If self-heal happens later, preserve them unchanged for regression.

### Step 3: Run the wiki-only answer pass

This is the blind retrieval phase. Use only `wiki/` content.

- Start from `wiki/wiki-index.md` and the target source page
- Navigate through linked wiki pages as needed
- Do not read `raw/` during this phase
- Answer each of the five questions as if you were a user relying on the wiki

For each question, record:

- `status`: `pass`, `partial`, or `fail`
- `pages_used`: the wiki pages required to answer it
- `page_count`: number of pages opened
- `answer`: a short answer grounded in the wiki

### Step 4: Grade the result

Use the following grading rules:

- `pass`: the wiki answer is correct, specific enough, and faithful to the source
- `partial`: the wiki captures the gist but misses a critical detail, condition, contrast, or example
- `fail`: the wiki cannot answer the question, answers incorrectly, or requires unsupported inference

Compute:

- `coverage_score = pass / 5`
- `partial_count`
- `fail_count`
- `avg_page_count`

Default acceptance threshold:

- At least `4/5` questions must pass
- Average `page_count` should be `<= 3`
- No failed question should be central to the source's main contribution

### Step 5: Diagnose gaps

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

### Step 6: Self-heal high-confidence gaps

Run this step automatically whenever the source does not meet the acceptance threshold, unless the user explicitly requested review-only mode.

Allowed self-heals:

- Add omitted source-grounded facts to an existing source page
- Add or repair cross-references between existing pages
- Downgrade an overstated relation from direct to extended when only synthesis support exists
- Remove or rewrite unsupported cross-references when the wiki graph is making retrieval less honest
- Tighten an overly abstract paragraph so it directly answers the missed question
- Create a missing concept or entity page when it is clearly central and well-supported
- Update `wiki/wiki-index.md` and `wiki/log.md` for any new or changed pages

Do not self-heal:

- Speculative interpretations
- Invented bridge claims used only to keep a weak link alive
- Broad conceptual rewrites
- Value judgments or strategic recommendations not explicit in the source
- Comparisons that require synthesis beyond high-confidence evidence

If a gap is important but not safe to self-heal:

- Create or update a page in `wiki/questions/`, or
- Mark the corresponding source in `raw/raw-index.md` as `update-needed`

### Step 7: Run same-question regression after self-heal

After patching, re-run the same five primary questions with the same wiki-only restriction.

This regression layer exists to answer one narrow question: did the repair fix the known gaps without breaking earlier coverage?

Do not use regression alone as the final `Ready` verdict after self-heal.

If regression fails, diagnose the remaining gaps, self-heal again if the gaps are high-confidence, and continue until one of the following is true:

- The source reaches the acceptance threshold
- Only low-confidence gaps remain
- Two repair cycles have completed

If the source still fails after two repair cycles, return a clear unresolved-gap report instead of continuing to patch indefinitely.

### Step 8: Run holdout eval

After a self-healed source passes same-question regression, generate exactly three new `holdout` questions.

Holdout questions must satisfy all of the following:

- They are explicitly supported by the same raw source
- They are materially different from the primary five questions
- They were not revealed during diagnosis, patch planning, or self-heal
- They are not trivial paraphrases of previously used questions

Run a fresh wiki-only answer pass for the holdout set with the same retrieval restrictions:

- Start from `wiki/wiki-index.md` or the target source page
- Use only `wiki/`
- Record `status`, `pages_used`, `page_count`, and `answer`

Grade holdout questions with the same `pass / partial / fail` rubric.

Final `Ready = yes` after self-heal requires both:

- Same-question regression meets the acceptance threshold
- Holdout score is at least `2/3 pass`
- No holdout failure is central to the source's main contribution

If holdout fails and the newly exposed gaps are high-confidence, self-heal once more, then rerun regression and a new holdout set, subject to the same two-cycle repair cap.

If no self-heal was needed because the initial primary pass already met the threshold, the primary pass may stand as the final verdict. Holdout is still recommended for stricter audits, but required whenever self-heal occurred.

### Step 9: Report

Present results in this format:

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
   - Pages: [[source-...]], [[concept-...]]
   - Gap: -
2. [question]
   - Status: fail
   - Pages: [[source-...]]
   - Gap: missing-fact

### Holdout Questions
1. [question]
   - Status: pass
   - Pages: [[source-...]]
   - Gap: -
2. [question]
   - Status: partial
   - Pages: [[concept-...]], [[source-...]]
   - Gap: too-abstract

### Actions
- Updated: [[source-...]], [[concept-...]]
- Created: [[question-...]]
- Deferred: [short note]

### Verdict
- Ready: yes | no
- Reason: one-line conclusion
```

If pages were edited, append to `wiki/log.md`:

```markdown
## [YYYY-MM-DD] coverage-review | Source title
- Questions evaluated: 5
- Initial score: 3/5
- Regression score: 5/5
- Holdout score: 2/3
- Updated pages: [[page1]], [[page2]]
- Created pages: [[page3]]
- Summary: one-line summary of the coverage gaps and repairs
```

## Rules

- Preserve phase separation: `raw/` is allowed for question generation and patch verification, but never for the wiki-only answer pass
- Prefer updating existing pages before creating new ones
- Every newly created page must have YAML frontmatter and at least one inbound link
- Use `[[wiki-link]]` references and explain relationships in related-page sections
- Do not call a source "covered enough" if users can only answer by reconstructing the source from scattered fragments
- Treat removal of a weak or false link as a successful repair when it makes the wiki more honest
- If you cannot state the bridge for a relation in one concrete sentence, do not preserve the link
- Default behavior is `review -> self-heal -> re-evaluate`; only skip self-heal when the user explicitly asks for review-only mode
- Same-question regression is a repair check, not proof of generalization
- After any self-heal, `Ready` must be based on both regression and holdout results
