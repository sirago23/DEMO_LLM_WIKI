# Runbook: Day-to-Day Operations

Reference for ongoing wiki maintenance and bot operations.

---

## Ingest a new source

1. Drop the source file into `raw/` (PDF, markdown, or text)
2. Tell Claude Code: `ingest raw/<filename>`
3. Claude Code follows the `llm-wiki-to-n8n` skill automatically:
   - Reads the source → extracts knowledge → alignment check with you
   - Writes wiki pages → commits + pushes to GitHub
   - Updates n8n `pages` array → validates workflow → exports JSON
   - Updates `prompts/faq-bot.system.md` → commits + pushes
4. Verify: open the Railway n8n URL → check last execution succeeded

**Rule:** One source file per ingest session. Do not batch multiple sources.

---

## Run a lint pass

Tell Claude Code: `run a lint pass` or `lint the wiki`.

Claude Code follows `workflows/lint.md` and checks:
- Orphan pages (no inbound links)
- Broken wikilinks
- Index completeness (every file has an index entry)
- Stale `updated` dates
- Missing cross-references
- Data gaps (from `wiki/overview.md`)

Lint fixes are committed and pushed in the same batch as other pending changes.

---

## Query the wiki

Tell Claude Code: `how do I <question>` or ask any FAQ question.

Claude Code follows `workflows/query.md`:
1. Reads `wiki/index.md`
2. Fetches relevant pages
3. Synthesizes an answer with `[[wikilink]]` citations
4. If the answer is valuable, files it back as a `wiki/faq/` page

---

## Add a new wiki page manually

Use the templates in `wiki/entities/_template.md`, `wiki/concepts/_template.md`, etc.

After adding:
1. Add an entry to `wiki/index.md` (update count + date)
2. Add cross-references from related pages
3. If the page should be in the bot's context, add it to the `pages` array in the n8n "Fetch wiki content" Code node
4. Commit + push

---

## Update the bot system prompt

Source of truth: `prompts/faq-bot.system.md`

1. Edit `prompts/faq-bot.system.md`
2. Copy the content between the `---` delimiters
3. Paste into the n8n AI Agent node's **System Message** field
4. Save the workflow in n8n
5. Export: `mcp__n8n__n8n_get_workflow` (mode: full) → overwrite `n8n/faq-bot.workflow.json`
6. Commit + push both files

---

## Update the n8n pages array

When new wiki pages should be added to the bot's knowledge:

1. Fetch current node: `mcp__n8n__n8n_get_workflow` with `nodeNames: ["Fetch wiki content"]`
2. Add new page paths to the `pages` array (no duplicates; sort: index → concepts → entities → faq)
3. Update via `mcp__n8n__n8n_update_partial_workflow`
4. Validate: `mcp__n8n__n8n_validate_workflow` → expect 0 errors, 0 warnings
5. Export full workflow → overwrite `n8n/faq-bot.workflow.json`
6. Update `prompts/faq-bot.system.md` → "Wiki pages fetched per query" list
7. Commit + push

**Never add `sources/*.md` pages** to the n8n fetch list — source pages are for human reference only.

---

## Verify the bot end-to-end

1. Send a test question via LINE (e.g., "What is your return policy?")
2. Check n8n execution log → confirm "Fetch wiki content" node ran and returned content
3. Confirm the reply arrived in LINE with a source citation

Test cases to cover:
- In-wiki question → answer with `Source:` citation
- SCG question with no wiki data → "Great question! I'm looking into this..."
- Off-topic question (e.g., "What's the weather?") → general answer, no Source line

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| Bot returns stale content after push | GitHub CDN cache (~5 min) | Wait and retry |
| n8n execution fails on HTTP fetch | GitHub raw URL changed or repo went private | Check URL; ensure repo is public |
| Bot doesn't reply to LINE messages | Workflow not active or webhook URL wrong | Activate workflow; re-verify webhook in LINE |
| Validation errors after n8n update | Malformed jsCode | Re-fetch node, diff the change, fix the array |
| `git push` fails | Remote has changes not in local | `git pull --rebase origin main` then push |

---

## Key references

| Resource | Location |
|----------|----------|
| Wiki index | `wiki/index.md` |
| Operation log | `wiki/log.md` |
| Bot system prompt | `prompts/faq-bot.system.md` |
| n8n workflow export | `n8n/faq-bot.workflow.json` |
| Ingest SOP | `workflows/ingest.md` |
| Lint SOP | `workflows/lint.md` |
| Query SOP | `workflows/query.md` |
| GitHub raw base | `https://raw.githubusercontent.com/sirago23/DEMO_LLM_WIKI/main/wiki/` |
| Railway n8n | `https://primary-production-da18a.up.railway.app` |
| n8n workflow ID | `TlLMB5bNCDP7PcXD` |
