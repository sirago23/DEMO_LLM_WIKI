# Demo_LLM_wiki

A persistent, compounding knowledge base built on the LLM Wiki pattern, with a LINE chatbot that answers FAQ questions from the wiki.

## How it works

1. Sources go into `raw/` (immutable — never modified by the LLM)
2. Claude Code reads sources, extracts knowledge, and writes/updates pages in `wiki/`
3. `wiki/` is pushed to this public GitHub repo
4. The LINE FAQ bot (n8n on Railway) reads `wiki/index.md` at query time, fetches the relevant pages, and answers with citations

## Quick links

- [CLAUDE.md](CLAUDE.md) — wiki schema and operating rules (read this first)
- [wiki/index.md](wiki/index.md) — full catalog of wiki pages
- [wiki/log.md](wiki/log.md) — chronological record of ingests and queries
- [workflows/ingest.md](workflows/ingest.md) — how to add a new source
- [workflows/query.md](workflows/query.md) — how the bot retrieves and answers
- [workflows/lint.md](workflows/lint.md) — how to health-check the wiki
- [prompts/faq-bot.system.md](prompts/faq-bot.system.md) — LINE bot system prompt (source of truth)
- [docs/](docs/) — setup guides (GitHub, Railway, LINE, runbook)

## Structure

```
wiki/           LLM-owned knowledge base (Claude writes, you read)
raw/            Immutable source documents (you add, LLM reads only)
workflows/      WAT SOPs — operating instructions for Claude Code
tools/          Optional Python helpers for batch operations
prompts/        Bot system prompts
n8n/            Exported workflow JSON + Railway instance metadata
docs/           Setup guides and runbook
```

## Ingest a new source

Drop a file into `raw/` and tell Claude Code: "ingest `raw/<filename>`". Claude Code will follow `workflows/ingest.md`.

## Setup

See [docs/setup-github.md](docs/setup-github.md), [docs/setup-railway-n8n.md](docs/setup-railway-n8n.md), [docs/setup-line-bot.md](docs/setup-line-bot.md).
