# Setup: LINE Messaging API

The FAQ bot receives messages via the LINE Messaging API and replies through n8n on Railway.

---

## 1. Create a LINE Messaging API channel

1. Go to [developers.line.biz](https://developers.line.biz) → Log in with your LINE account
2. Create a **Provider** (e.g., "Seattle Coffee Gear")
3. Create a new channel → **Messaging API**
4. Fill in: channel name, description, category, subcategory
5. Agree to terms → Create

## 2. Get your Channel Access Token

1. Open your channel → **Messaging API** tab
2. Scroll to **Channel access token (long-lived)**
3. Click **Issue** → copy the token
4. Store it in n8n as an HTTP Header Auth credential (see [setup-railway-n8n.md](setup-railway-n8n.md))

## 3. Set the webhook URL

1. In your LINE channel → **Messaging API** tab → Webhook settings
2. Set Webhook URL to:
   ```
   https://<your-railway-domain>/webhook/faq-bot
   ```
3. Click **Verify** — you should get a success response (n8n workflow must be active)
4. Enable **Use webhook** toggle

## 4. Disable auto-reply and greeting messages

In LINE channel → **Messaging API** tab:
- **Auto-reply messages**: Off
- **Greeting messages**: Off (optional — the bot handles its own replies)

## 5. Add the bot to LINE

1. In **Messaging API** tab → **Bot basic ID** or **QR code**
2. Scan the QR code with LINE to add the bot as a friend
3. Send a test message — you should receive a reply from the n8n workflow

## 6. Store credentials safely

| Secret | Where to store |
|--------|---------------|
| Channel Access Token | n8n credential store only |
| Channel Secret | n8n credential store only (if using webhook verification) |

Never commit these to git. The `.env.example` has a `LINE_CHANNEL_TOKEN` placeholder for local testing scripts only.

## Troubleshooting

| Symptom | Check |
|---------|-------|
| Webhook verify fails | n8n workflow is active; Railway URL is correct; no trailing slash |
| Bot doesn't reply | Check n8n execution log for errors; confirm LINE credential is set |
| Bot replies twice | Disable LINE auto-reply in channel settings |
