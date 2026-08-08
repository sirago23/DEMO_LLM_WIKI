# Setup: GitHub Repository

The wiki is served from a **public** GitHub repository. The LINE bot fetches pages directly from GitHub raw URLs — no auth required.

---

## 1. Create the repository

1. Go to github.com → New repository
2. Name: `DEMO_LLM_WIKI` (or any name — update the raw base URL everywhere if you change it)
3. Set visibility to **Public** (required — raw URLs are unauthenticated)
4. Do not initialize with README (the local repo already has one)

## 2. Push the local repo

```bash
git remote add origin https://github.com/<your-username>/DEMO_LLM_WIKI.git
git branch -M main
git push -u origin main
```

## 3. Confirm the raw URL works

Open this URL in a browser (unauthenticated):

```
https://raw.githubusercontent.com/<your-username>/DEMO_LLM_WIKI/main/wiki/index.md
```

You should see the raw markdown of `wiki/index.md`. If it 404s, check that the repo is public and the push succeeded.

## 4. Update hardcoded references

If your username or repo name differs from `sirago23/DEMO_LLM_WIKI`, update these locations:

| File | What to change |
|------|----------------|
| `CLAUDE.md` | GitHub raw base URL |
| `prompts/faq-bot.system.md` | Architecture note |
| `n8n` Code node `jsCode` | `baseUrl` constant |
| `wiki/sources/*.md` | "Original" field links |

## 5. Git workflow (ongoing)

After every ingest, Claude Code runs:

```bash
git add wiki/
git diff --cached          # scan for secrets before committing
git commit -m "ingest: <source> — <N> pages created/updated"
git push origin main
```

A second commit follows after the n8n sync:

```bash
git commit -m "sync: n8n + system prompt after <source> ingest"
git push origin main
```

## Secret-safety rules

- This repo is **public** — never commit secrets
- `.env` and `.mcp.json` are gitignored
- Use `.env.example` and `.mcp.json.example` as templates
- n8n credentials live in the n8n credential store only
- Scan `git diff --cached` before every commit

## GitHub CDN caching

GitHub raw URLs may be cached for ~5 minutes after a push. If the bot returns stale content immediately after a push, wait a few minutes and retry.
