# Content Rules

## Files and Pages

- Files under `<domain>/raw/` are immutable. The one exception is `<domain>/raw/raw-index.md`, which is updated after ingest (format in `page-formats.md` → `<domain>/raw/raw-index.md`).
- Use kebab-case filenames.
- Source page filenames are topic-oriented: name a `<domain>/wiki/sources/` page after the paper's core method or concept, with no venue or year prefix. Example: `source-graph-information-bottleneck` (✅), not `source-neurips-2020-graph-information-bottleneck` (❌). The reader recognizes the topic from the filename alone.
- A wiki filename must be unique inside its domain. For duplicates across domains, see Links.
- Every wiki content page has YAML frontmatter (`page-formats.md`) and at least one inbound link.

## Links

- Link with Obsidian wiki links: `[[page-name]]`.
- A `[[link]]` inside a domain targets only pages of the same domain. Domains do not link to each other.
- `index.md` is the one exception: it links to each domain's overview with a path link, e.g. `[[deep-learning-research/wiki/overview|deep-learning-research]]`, because every domain has an `overview.md`.
- Obsidian resolves a short `[[page-name]]` vault-wide. When two domains hold the same filename, the link can resolve to the wrong domain, so lint reports such duplicates.

## Log

Each domain keeps its own `<domain>/wiki/log.md`. It records substantive wiki content changes in that domain only. Tooling and documentation maintenance stay out of it. It has no frontmatter and no `[[links]]`, and it stays outside the semantic graph. At the start of each year, move the previous year's entries into `<domain>/wiki/log-YYYY.md`. Archives follow the same rules as the log.

## Domains

A domain answers "which large area am I in?" A `subcluster` answers "which topic line inside that domain should I enter first?"

- A domain is a root folder that contains `GUIDE.md` (`repository-structure.md` → Identifying Domains). A page belongs to the domain whose folder holds it, so frontmatter carries no domain field.
- `subcluster` is optional and singular. Use it when the page clearly belongs to one stable topic entrance inside its domain. Valid keys live in `<domain>/GUIDE.md` → Subcluster Keys.
- Domains and subclusters are navigation aids. `[[links]]` still carry the semantic relations.

### GUIDE.md precedence

Each `<domain>/GUIDE.md` (format in `page-formats.md` → `<domain>/GUIDE.md`) sets the domain's scope, material type, and subcluster keys. Its optional sections override defaults for that domain only:

| GUIDE.md section | Default it overrides |
|------------------|----------------------|
| `## Analysts` | The default analyst team in `workflows.md` → Ingest |
| `## Source Page` | The source-page structure in the compile skill, W2 |
| `## Coverage Questions` | The question types in the coverage-review skill, Step 2 |

GUIDE.md cannot override the shared framework: frontmatter, link rules, the log, the raw-index format, the phase contract, and blind-review separation. When GUIDE.md conflicts with any of these, `rules/` wins.

## Relations

- Link only with high confidence: one concrete sentence, supported by raw material or existing wiki content, must justify the relation. When a relation is plausible but not yet supportable, capture it as a `<domain>/wiki/questions/` page instead.
- Label every relation as one of two kinds:
  - `直接關聯` — an explicit source statement, dependency, comparison, author/work relation, or question/source relation.
  - `延伸關聯` — editorial synthesis already grounded elsewhere in the wiki.
- Write related-page entries as `- [[page-name]] — 直接關聯：<the concrete bridge>`. The note names the actual bridge, so catch-alls such as `related` or `see also` never qualify.
- Every technical term resolves at its first appearance, per `writing-style.md` → First-Appearance Terms.

## Creating Subclusters and Domains

Default to an existing subcluster. Handle a secondary angle with links and relation notes. When the fit is ambiguous, leave `subcluster` empty until a stable topic entrance emerges.

Create a subcluster when all of these hold:

1. At least `3` strongly related pages exist, or more are clearly expected soon.
2. The topic has a clear entrance question and boundary inside its domain.
3. Readers benefit from entering through a topic page instead of scanning a long flat domain overview.

After creating a subcluster:

1. Add the key and its definition to `<domain>/GUIDE.md` → Subcluster Keys.
2. Create the entrance page under `<domain>/wiki/subclusters/`, with scope, boundary, and representative pages.
3. Link the new entrance from `<domain>/wiki/overview.md`.
4. Reassign existing pages only where it improves navigation.

Agents never create a domain on their own. A new domain starts only when the user asks for it, and the agent designs its `GUIDE.md` together with the user. To create a domain:

1. Create the folder `<domain>/` at the repository root, named with the kebab-case domain key.
2. Write `<domain>/GUIDE.md` with the user: Scope, Material, Subcluster Keys, and any override sections.
3. Create `<domain>/raw/raw-index.md` with the table header only.
4. Create `<domain>/wiki/overview.md` and an empty `<domain>/wiki/log.md`.
5. Add the domain to `index.md` with a path link to its overview.
