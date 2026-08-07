# Workflow: Answer a Query

**Objective:** Answer a user question using wiki content, with citations. Optionally file the
answer back as a FAQ page.

**Trigger:** User asks a question about the domain.

**Required inputs:**
- User question

**Expected outputs:**
- Answer with citations to wiki pages
- Optionally: new `wiki/faq/<slug>.md` page (if answer is valuable)
- Optionally: updated `wiki/index.md` and `wiki/log.md`

---

## Steps

### 1. Read `wiki/index.md`
Scan all page summaries and tags to identify which pages are likely relevant to the question.
Do not skip this step — the index is designed to make retrieval fast.

### 2. Read relevant pages
Read the candidate pages identified in step 1. Read 2–5 pages maximum for most questions.
If a page links to another relevant page via `[[wikilink]]`, read that too.

### 3. Synthesize the answer
- Ground your answer entirely in wiki content. Do not add information from training data.
- If the wiki does not contain enough information, say so explicitly: "The wiki doesn't cover
  this yet. Consider adding a source on this topic."
- Cite sources using the wikilink format: `[[entities/plan-pro]]`

### 4. Decide whether to file the answer
File the answer as a FAQ page if:
- The question is a natural FAQ (likely to be asked again by LINE users)
- The answer required synthesizing 2+ pages (so the synthesis is worth preserving)
- The answer revealed a gap or connection not already in the wiki

If filing:
- Create `wiki/faq/<question-slug>.md` from the FAQ template
- Update `wiki/index.md` (add to FAQ table)
- Append to `wiki/log.md`

### 5. Log (if you filed a FAQ page)
```
## [YYYY-MM-DD] query | <question>
- Filed answer to: faq/<slug>
- Pages read: index, <list>
```

---

## For the LINE bot (n8n AI Agent)

The bot follows the same logic, automated:
1. Receive user message (text)
2. Fetch `wiki/index.md` from GitHub raw
3. Identify relevant pages from index
4. Fetch those pages using `get_wiki_page(filename)` tool
5. Synthesize answer using system prompt rules
6. Reply via LINE reply endpoint

The bot does **not** file FAQ pages back — that is a Claude Code operation only.
