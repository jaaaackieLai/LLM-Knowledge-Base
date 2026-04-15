---
name: language
description: Use when the user asks to change the repository's configured wiki language and update the language slot consistently across non-wiki maintenance documents, skills, and slash commands.
---

# Configure Repository Wiki Language

Update the repository's configured wiki language across maintenance documentation and operational instructions, including both user communication language and wiki-writing language.

## Input

Use the user's request text to determine the target language.

- Require a target language, such as `english`, `zh-TW`, or `japanese`
- If no target language is provided, ask the user which language should become the configured wiki language

## Workflow

### Step 1: Resolve the target language

- Read the requested language from the user's input
- Normalize it into the exact wording that should appear in repository documentation
- Treat this as the repository's configured language for future user communication and wiki writing

### Step 2: Update the language slots

Update the configured language slots for both user communication and wiki writing consistently across non-wiki maintenance files, including:

- `AGENTS.md`
- `CLAUDE.md`
- `rules/writing-style.md`

Replace wording such as:

- `For this repository, use zh-TW.`
- `For this repository, communicate with the user in zh-TW and write wiki content in zh-TW.`
- `the configured wiki language is zh-TW`
- `in this repository, use zh-TW`

with the new target language.

### Step 3: Adjust language-dependent wording

- Keep repository documentation in English unless the user explicitly asks to change that policy too
- Update wording or examples that depend on the configured language for user communication or wiki writing
- Prefer generic phrasing whenever possible so future language changes require fewer edits

### Step 4: Validate

- Search non-wiki maintenance files for stale references to the old configured wiki language or old wiki-writing language
- Confirm the new language is reflected consistently across documentation, skills, slash commands, and wiki-writing instructions

## Rules

- Do not rewrite existing `wiki/` content pages as part of this workflow
- Do not modify files under `raw/`, except if the user separately asks for a tracking-table update
- Treat this as a repository-configuration update, not a semantic content rewrite
- Keep the workflow reusable for future language changes
