# Plan: LLM Wiki Knowledge Base + LINE FAQ Chatbot

## Context

Instantiating the LLM Wiki pattern as a persistent, compounding FAQ knowledge base, with a LINE bot
that answers user questions by reading the wiki at query time.

**Locked decisions:**
- Delivery path: n8n → public GitHub repo (raw URLs, no auth needed)
- Retrieval: index.md-first (Karpathy style) via n8n AI Agent + `get_wiki_page` HTTP tool
- Domain: Business / product FAQ
- Answer LLM: Claude (Anthropic)
- n8n: separate new Railway instance (isolated from TrustmeBro)
- Ingest actor: Claude Code locally — user drops source, I write/update wiki pages and push

## Architecture

**A. Wiki (local → public GitHub)** — Claude Code maintains this.
**B. LINE FAQ bot (Railway n8n)** — reads wiki from GitHub at query time.

### Bot data flow
```
LINE user message
  → LINE platform → n8n Webhook /webhook/faq-bot
  → Code: extract { text, replyToken, userId }
  → HTTP GET wiki/index.md from GitHub (raw URL)
  → AI Agent (Claude) with get_wiki_page tool
       reads index → fetches relevant pages → synthesizes answer + citations
  → Code: format for LINE
  → HTTP POST LINE reply endpoint
```

GitHub raw base: `https://raw.githubusercontent.com/<user>/Demo_LLM_wiki/main/wiki/`

## Steps

| # | Step | Complexity | Time | Gate |
|---|------|-----------|------|------|
| 0 | Scaffold structure + this file | Low | 10 min | — |
| 1 | CLAUDE.md schema | Medium | 30–45 min | — |
| 2 | Install session-handoff skill globally | Low | 15 min | — |
| 3 | Seed wiki (index, log, templates) | Low–Med | 20–30 min | — |
| 4 | WAT workflow SOPs | Medium | 30 min | — |
| 5 | Ingest first real source | Medium | 20–30 min | User provides source ⚑ |
| 6 | GitHub repo + push | Low | 15 min | User confirms account ⚑ |
| 7 | New Railway n8n instance | Medium | 30–45 min | User deploys ⚑ |
| 8 | Build bot workflow node by node | High | 1.5–2 hr | Confirm each node ⚑ |
| 9 | LINE Messaging API wiring | Medium | 20–30 min | User sets webhook ⚑ |
| 10 | Prompt tuning | Medium | 30–45 min | — |
| 11 | Docs, runbook, lint pass | Low–Med | 30 min | — |

**Total:** ~6–8 hrs across sessions.

## Node-by-node build order (Step 8)
1. Webhook only → confirm raw LINE payload arrives
2. Code (extract) → confirm `{ text, replyToken, userId }`
3. HTTP (LINE reply) with hardcoded echo → confirm reply reaches phone
4. HTTP (fetch index.md from GitHub) → confirm index content arrives
5. AI Agent + Anthropic Chat Model (no tool) → confirm answer text
6. Add `get_wiki_page` HTTP tool to agent → confirm page fetch in logs
7. Wire agent answer to LINE reply → confirm end-to-end

## Secret-safety rules
- Repo is **public** — zero secrets committed. `.env`, `.mcp.json` are gitignored.
- n8n secrets live in n8n credential store only.
- Scan `git diff --cached` before every commit.

## Verification
1. `wiki/index.md` links resolve; log parseable with `grep "^## \[" wiki/log.md | tail -5`
2. Raw GitHub URL for `wiki/index.md` loads unauthenticated in browser
3. `health_check` (n8n-mcp) passes for new Railway instance
4. LINE question → answer with citation; unknown question → "I don't know"
5. n8n execution log shows `get_wiki_page` calls for correct pages
6. `session-handoff` skill appears globally and produces handoff template
