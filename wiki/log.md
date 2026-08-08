# Wiki Log

Append-only chronological record of all wiki operations.
Parse with: `grep "^## \[" wiki/log.md | tail -5`

---

## [2026-08-08] ingest | Refund Policy — Seattle Coffee Gear
- Source: `raw/Refund policy _ Seattle Coffee Gear.pdf`
- Pages created: sources/refund-policy-seattle-coffee-gear, concepts/refund-policy, concepts/return-process, concepts/like-new-condition
- Pages updated: concepts/damaged-missing-merchandise (added replacement return window), overview (added returns theme, removed resolved gaps), index (count 7→11, added 4 pages)
- Key additions: 30-day return window (retail from purchase, online from ship date), Open Box Final Sale rule, RMA process (email support@seattlecoffeegear.com), Like New condition standard, 5% restocking fee >$1,000, consumables never eligible, self-installed PID voids eligibility, Miele requires 1-800-999-1360 first, extended holiday window 11/1–12/24/2025 → eligible until 1/23/2026

## [2026-08-08] update | System prompt — off-topic and no-data handling rules added
- Workflow: n8n TlLMB5bNCDP7PcXD (Demo_Chatbot_LLM_WIKI)
- Change: Added two new answer rules to FAQ Agent systemMessage:
  - Rule 4: SCG-related question but no wiki data → "Great question! I'm looking into this for you. Our team will get back to you with the details as soon as possible."
  - Rule 5: Clearly off-topic question (jokes, weather, trivia) → answer helpfully using general knowledge in a friendly retail assistant tone; no Source citation
- Tested all three cases (executions 3415–3417): in-wiki ✅, no-data ✅, off-topic ✅
- Source of truth updated: prompts/faq-bot.system.md

## [2026-08-07] update | Wiki initialized
- Created wiki structure: index.md, log.md, overview.md, template pages
- Status: empty — awaiting first source ingest

## [2026-08-08] update | FAQ bot architecture — pre-fetch replaces tool calling
- Workflow: n8n TlLMB5bNCDP7PcXD (Demo_Chatbot_LLM_WIKI)
- Change: Removed get_wiki_page tool node + Fetch index.md HTTP node. Added "Fetch wiki content" Code node (typeVersion 1) that fetches all 6 wiki pages sequentially via this.helpers.httpRequest and injects content into the FAQ Agent prompt.
- Reason: Gemini 2.5 Flash consistently failed to call tools in this n8n setup (single 787ms API call, tool node showed itemsInput: 0). Pre-fetch eliminates tool-calling dependency entirely.
- All 8 test questions passed (executions 3403–3411). Bot answers correctly with citations.
- Pages in bot context: index.md, concepts/shipping-policy, concepts/order-cancellation-policy, concepts/damaged-missing-merchandise, entities/beanz, entities/safonia-shipping-protection
- Exported: n8n/faq-bot.workflow.json

## [2026-08-07] ingest | Shipping Policy — Seattle Coffee Gear
- Source: `raw/Shipping policy _ Seattle Coffee Gear.pdf`
- Pages created: sources/shipping-policy-seattle-coffee-gear, concepts/shipping-policy, concepts/order-cancellation-policy, concepts/damaged-missing-merchandise, entities/beanz, entities/safonia-shipping-protection
- Pages updated: overview, index
- Key additions: free shipping threshold ($49+), no-change-after-tracking rule, Beanz™ immediate-lock rule, address change restrictions, refused delivery fee (20% + original shipping), 5-day damage/missing reporting window, open box final sale rule
