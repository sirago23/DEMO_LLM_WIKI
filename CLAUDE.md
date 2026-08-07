# Wiki Schema & Operating Rules

This is the schema document for the LLM Wiki knowledge base. It defines conventions, page
types, file formats, and operating workflows. Read this before touching any wiki page.

---

## The WAT Architecture

This project follows the WAT framework (Workflows, Agents, Tools):

- **Workflows** (`workflows/`) — Markdown SOPs that define what to do step-by-step
- **Agent** — Claude Code. Reads workflows, makes decisions, calls tools, updates wiki
- **Tools** (`tools/`) — Python scripts for deterministic execution (build only when needed)

**Rule:** For any non-trivial operation (ingest, query, lint), read the relevant workflow SOP
before acting. The SOP is the source of truth for that operation.

---

## Directory Layout

```
wiki/           LLM-owned knowledge base. Claude writes; user reads.
  index.md      Content catalog — READ FIRST on every query
  log.md        Append-only chronological log of all operations
  overview.md   Top-level synthesis of the knowledge domain
  entities/     Pages for specific things: products, plans, features, teams
  concepts/     Pages for policies, processes, how-tos, abstract topics
  faq/          Curated Q&A pages (answers filed back from queries)
  sources/      Per-source summaries with key takeaways

raw/            Immutable source documents. LLM reads; never modifies.
  assets/       Images referenced by wiki pages (download locally)

workflows/      SOPs that govern wiki operations
tools/          Python helpers (build only when a task is too repetitive to do by hand)
prompts/        Bot system prompts (source of truth for bot behavior)
n8n/            Exported workflow JSON + instance metadata
docs/           Setup guides and operational runbook
```

**Ownership rule:** Claude owns `wiki/`. The user owns `raw/`. Nobody modifies the other's layer.

---

## Page Types

Every page in `wiki/` (except `index.md` and `log.md`) must start with YAML frontmatter.

### Frontmatter fields

```yaml
---
title: "Human-readable page title"
type: entity | concept | source | faq | overview
tags: [tag1, tag2]          # lowercase, hyphenated
updated: YYYY-MM-DD         # date last meaningfully changed
sources: [filename1, url1]  # raw sources this page draws from
---
```

### Entity page (`wiki/entities/`)
A page about a specific named thing: a product, a pricing plan, a feature, a team, an integration.

Structure:
```markdown
---
title: "Product Name"
type: entity
tags: [product, pricing]
updated: YYYY-MM-DD
sources: [source-file.md]
---

# Product Name

One-sentence description.

## Key facts
- Fact 1
- Fact 2

## Details
Prose explanation.

## Related
- [[concepts/related-concept]]
- [[entities/related-entity]]
```

### Concept page (`wiki/concepts/`)
A page about a policy, process, how-to, or abstract topic — not a specific named thing.

Structure mirrors entity pages. Use ## Steps for processes, ## Rules for policies.

### Source page (`wiki/sources/`)
A summary of a single source document in `raw/`.

```markdown
---
title: "Source: <document name>"
type: source
tags: [source, <domain-tag>]
updated: YYYY-MM-DD
sources: [<raw-filename>]
---

# Source: <document name>

**Original:** `raw/<filename>` | **Ingested:** YYYY-MM-DD

## Summary
2–4 sentence summary of what this source covers.

## Key takeaways
- Takeaway 1
- Takeaway 2

## Pages updated by this source
- [[entities/...]]
- [[concepts/...]]
```

### FAQ page (`wiki/faq/`)
A curated Q&A page, either pre-written or filed back from a query.

```markdown
---
title: "Q: <question>"
type: faq
tags: [faq, <topic>]
updated: YYYY-MM-DD
sources: []
---

# Q: <question>

**A:** Direct answer in 1–3 sentences.

## Detail
Supporting explanation.

## Related
- [[entities/...]]
- [[concepts/...]]
```

### Overview page (`wiki/overview.md`)
The top-level synthesis document. No frontmatter section — just content.
Updated by ingest and lint passes.

---

## Cross-referencing (Wikilinks)

Use `[[path/to/page]]` syntax for all internal links. Path is relative to `wiki/`.

Examples:
- `[[entities/product-basic-plan]]`
- `[[concepts/refund-policy]]`
- `[[faq/how-to-upgrade]]`

Omit the `.md` extension in wikilinks. Obsidian resolves them correctly.

**Cross-reference rule:** Every page must link to at least one other wiki page.
Orphan pages (no inbound links) are flagged in lint passes.

---

## index.md Format

`wiki/index.md` is the master catalog. The bot reads it first on every query to decide which
pages to fetch. Keep it current — update it on every ingest and whenever a page is added.

```markdown
# Wiki Index

Last updated: YYYY-MM-DD | Pages: N

## Entities
| Page | Summary | Tags |
|------|---------|------|
| [[entities/product-name]] | One-line description | product, pricing |

## Concepts
| Page | Summary | Tags |
|------|---------|------|
| [[concepts/refund-policy]] | Refund rules and timelines | policy, billing |

## FAQ
| Page | Summary | Tags |
|------|---------|------|
| [[faq/how-to-upgrade]] | Steps to upgrade a plan | billing, account |

## Sources
| Page | Summary | Ingested |
|------|---------|----------|
| [[sources/product-docs-v1]] | Product documentation v1 | YYYY-MM-DD |
```

---

## log.md Format

`wiki/log.md` is append-only — never delete entries.

Each entry starts with `## [YYYY-MM-DD] <type> | <title>` so it's grep-parseable.

Types: `ingest`, `query`, `lint`, `update`

```markdown
## [2026-08-07] ingest | Product Docs v1
- Source: `raw/product-docs-v1.md`
- Pages created: entities/plan-basic, entities/plan-pro, concepts/refund-policy
- Pages updated: overview
- Key additions: pricing tiers, refund window, contact info

## [2026-08-07] query | How do I upgrade my plan?
- Filed answer to: faq/how-to-upgrade
- Pages read: index, entities/plan-pro, concepts/upgrade-process
```

---

## Naming Conventions

- All filenames: lowercase, hyphenated, no spaces → `refund-policy.md`
- Entity pages: named after the entity → `entities/plan-basic.md`
- Concept pages: named after the concept → `concepts/refund-policy.md`
- Source pages: named after the source → `sources/product-docs-v1.md`
- FAQ pages: short question slug → `faq/how-to-upgrade.md`

---

## Ingest Operation

When the user asks to ingest a source, follow `workflows/ingest.md`. Summary:
1. Read the source in `raw/`
2. Discuss key takeaways with the user
3. Write a `sources/` summary page
4. Create or update entity/concept pages touched by this source
5. Update `wiki/index.md` with new/changed entries
6. Append an entry to `wiki/log.md`
7. Update `wiki/overview.md` if the synthesis changes

A single source may touch 5–15 wiki pages. That's normal.

---

## Query Operation

When the user asks a question, follow `workflows/query.md`. Summary:
1. Read `wiki/index.md` — find relevant page candidates
2. Read those pages
3. Synthesize an answer with citations (`[[page-path]]` format)
4. If the answer is valuable, file it back as a `faq/` page and update `index.md` + `log.md`

---

## Lint Operation

Periodically follow `workflows/lint.md`. Checks:
- Orphan pages (no inbound links)
- Stale claims superseded by newer sources
- Pages mentioned in `index.md` that don't exist
- Important concepts mentioned inline but lacking their own page
- Missing cross-references

---

## Secret-Safety Rules

- This repo is **public**. Zero secrets in committed files.
- `.env` and `.mcp.json` are gitignored — use `.env.example` and `.mcp.json.example` instead.
- n8n workflow secrets live in the n8n credential store, never in committed JSON.
- Scan `git diff --cached` before every commit. If a secret lands in git, rotate it immediately.

---

## n8n Bot Context

The LINE FAQ bot (n8n on Railway) does not write to the wiki. It is read-only.

- It fetches `wiki/index.md` from GitHub raw at query time
- It uses the `get_wiki_page` tool to fetch individual pages from GitHub raw
- It answers only from wiki content — no hallucination
- GitHub raw URL format: `https://raw.githubusercontent.com/<user>/<repo>/main/wiki/<path>`
- Note: GitHub CDN may cache raw files for ~5 minutes after a push

Bot system prompt source of truth: `prompts/faq-bot.system.md`
