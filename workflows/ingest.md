# Workflow: Ingest a Source

**Objective:** Read a new source document and integrate its knowledge into the wiki.

**Trigger:** User says "ingest `raw/<filename>`" or drops a file and asks me to process it.

**Required inputs:**
- Path to the source file (in `raw/`)

**Expected outputs:**
- New `wiki/sources/<slug>.md` summary page
- New or updated entity/concept pages
- Updated `wiki/index.md`
- New entry appended to `wiki/log.md`
- Updated `wiki/overview.md` if the synthesis changes

---

## Steps

### 1. Read the source
Read the full content of `raw/<filename>`.
If it contains images, note the image paths — read them separately after the text pass.

### 2. Identify key information
Extract:
- Named things (products, plans, features, services, teams) → will become entity pages
- Policies, processes, how-tos → will become concept pages
- Contradictions or updates to existing wiki claims → flag for update

### 3. Discuss with user (brief)
Surface the 3–5 most important takeaways. Ask if anything should be emphasized or de-emphasized.
Keep this brief — the goal is alignment, not a full review.

### 4. Write the source summary page
Create `wiki/sources/<source-slug>.md` using the source template.
- Slug: lowercase, hyphenated version of the source name
- Include: summary, key takeaways, list of pages to be updated

### 5. Create or update entity/concept pages
For each named thing or policy/process identified in step 2:
- If the page doesn't exist → create it from the appropriate template
- If it exists → read it, then update with new/changed information
- Add cross-references to related pages

A single ingest typically touches 5–15 pages. That is normal.

### 6. Update `wiki/index.md`
- Add new pages to the appropriate table (Entities, Concepts, Sources, FAQ)
- Update summaries for changed pages
- Update the "Last updated" date and page count

### 7. Append to `wiki/log.md`
Format:
```
## [YYYY-MM-DD] ingest | <source name>
- Source: `raw/<filename>`
- Pages created: <list>
- Pages updated: <list>
- Key additions: <brief note on what changed>
```

### 8. Update `wiki/overview.md`
If the ingest adds a new theme, changes the domain picture, or reveals a gap — update overview.
If nothing material changed at the overview level, skip this step.

---

## Edge cases

**Source contradicts existing wiki claim:** Note the contradiction in both pages. Add a
"Conflicts with" bullet. Ask the user which source is authoritative before updating.

**Source is very large:** Ingest in sections. Create the source summary first, then work
through entity/concept pages in batches. Append a partial log entry and update it when done.

**Source contains only images:** Read images directly. Write what you extract as the source
summary. Create entity/concept pages normally.

**Duplicate source:** If a source with overlapping content already exists, note the overlap in
both source pages and only update wiki pages that changed.
