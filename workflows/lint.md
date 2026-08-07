# Workflow: Lint the Wiki

**Objective:** Health-check the wiki for consistency, completeness, and freshness.

**Trigger:** User asks for a lint pass, or periodically after several ingests.

**Required inputs:** None (reads the full wiki).

**Expected outputs:**
- List of issues found, grouped by type
- Fixes applied (with user approval for non-trivial changes)
- Updated `wiki/log.md` with lint entry

---

## Checks to perform

### 1. Orphan pages
Find pages in `wiki/` that are not linked from any other page (no inbound `[[wikilink]]` references).
Action: Add links from related pages, or ask user whether to delete the orphan.

### 2. Broken wikilinks
Find `[[wikilink]]` references that don't resolve to an actual file.
Action: Fix the path or create the missing page.

### 3. Index completeness
Every file in `wiki/entities/`, `wiki/concepts/`, `wiki/faq/`, `wiki/sources/` should have an
entry in `wiki/index.md`. Find any that are missing.
Action: Add missing entries to the index.

### 4. Stale claims
Review entity/concept pages whose `updated` date is older than the most recent source on the
same topic. Flag pages that may have been superseded.
Action: Note "may be stale — verify against `[[sources/newer-source]]`" on the page.

### 5. Missing pages
Find important concepts or entities mentioned inline in wiki pages but lacking their own page.
(e.g., "See our Enterprise plan" with no `[[entities/plan-enterprise]]` page)
Action: Create a stub page or ask user to provide a source.

### 6. Missing cross-references
Find pages with no `## Related` section, or pages that should link to each other but don't.
Action: Add cross-references.

### 7. Data gaps
Look at `wiki/overview.md` for noted gaps. Suggest new sources that could fill them.
Action: Report suggestions to user — no automatic changes.

---

## Log entry
```
## [YYYY-MM-DD] lint | Full lint pass
- Orphans found: N (fixed M)
- Broken links: N (fixed M)
- Index gaps: N (fixed M)
- Stale claims flagged: N
- Suggested new sources: <list>
```
