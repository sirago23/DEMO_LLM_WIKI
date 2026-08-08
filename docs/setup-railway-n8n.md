# Setup: Railway n8n Instance

The LINE FAQ bot runs on **n8n deployed on Railway**. This guide covers deploying a fresh instance and importing the workflow.

---

## 1. Deploy n8n on Railway

1. Go to [railway.app](https://railway.app) → New Project → Deploy from Template
2. Search for **n8n** and select the official template
3. Railway will provision a PostgreSQL database and deploy n8n automatically
4. Once deployed, note your Railway app URL (e.g., `https://primary-production-xxxx.up.railway.app`)

## 2. Set environment variables in Railway

In your Railway project → Variables, set:

| Variable | Value |
|----------|-------|
| `N8N_HOST` | Your Railway domain (auto-set by template) |
| `WEBHOOK_URL` | `https://<your-railway-domain>/` |
| `N8N_ENCRYPTION_KEY` | Random 32-char string (keep secret) |

The n8n template usually sets these automatically. Verify in Settings → Variables.

## 3. Open n8n and create credentials

Navigate to your Railway URL → open n8n UI → Credentials → Add:

### Anthropic credential
- Type: **Anthropic**
- API Key: your Anthropic API key (from console.anthropic.com)

### LINE credential
- Type: **HTTP Header Auth** (or n8n built-in LINE credential if available)
- Header: `Authorization`
- Value: `Bearer <your-line-channel-access-token>`

## 4. Import the workflow

1. In n8n → Workflows → Import from file
2. Select `n8n/faq-bot.workflow.json` from this repo
3. After import, open the workflow and update credential references:
   - FAQ Agent node → Anthropic credential
   - LINE reply HTTP node → LINE credential
4. Activate the workflow (toggle in top-right)

## 5. Note the webhook URL

The webhook node listens at:

```
https://<your-railway-domain>/webhook/faq-bot
```

You'll need this URL when setting up the LINE Messaging API webhook (see [setup-line-bot.md](setup-line-bot.md)).

## 6. Verify the instance is healthy

Using n8n-mcp or the n8n UI:

- MCP: `mcp__n8n__n8n_health_check` → expect `{ status: "ok" }`
- UI: Settings → About → confirm version and DB connection

## 7. Update .mcp.json with your Railway URL

`.mcp.json` (gitignored) connects Claude Code's n8n MCP tools to your Railway instance:

```json
{
  "mcpServers": {
    "n8n": {
      "command": "npx",
      "args": ["-y", "n8n-mcp"],
      "env": {
        "N8N_API_URL": "https://<your-railway-domain>",
        "N8N_API_KEY": "<your-n8n-api-key>"
      }
    }
  }
}
```

Generate an n8n API key in n8n UI → Settings → API → Create API Key.

## Current instance

| Field | Value |
|-------|-------|
| Railway URL | `https://primary-production-da18a.up.railway.app` |
| Workflow ID | `TlLMB5bNCDP7PcXD` |
| Workflow name | `Demo_Chatbot_LLM_WIKI` |
| Webhook path | `/webhook/faq-bot` |
