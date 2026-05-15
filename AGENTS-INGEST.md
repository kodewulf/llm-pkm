# Ingest Module

This module is loaded when the user sends a message beginning with `ingest:`, or when the user asks to process or ingest a file without using a prefix (natural language fallback). Follow the workflow below exactly.

The filename to ingest is taken from the prefix argument (e.g. `ingest: raw/article.md`) or inferred from the user's message.

---

## Phase 1 — Pre-read checks (cheap; no full source read yet)

### Step 0 — Sidecar Detection

Before anything else, check whether the incoming file is a non-markdown binary (PDF, image, etc.):

- **File is markdown (`.md`)** → proceed directly to the Duplicate Check below. Frontmatter is read from the file itself.
- **File is non-markdown** → look for a sidecar at `raw/<filename>.meta.md` (e.g. `my-paper.pdf` → `my-paper.pdf.meta.md`).
  - **Sidecar found** → read the sidecar's frontmatter instead of the binary file's. Treat the sidecar as the identity document for all routing and duplicate detection steps below. Read the binary file itself only in Phase 2 for content extraction.
  - **Sidecar not found** → leave the binary file in `raw/` untouched. Append a `blocked` entry to today's log file noting that no sidecar was found and one must be created. Stop.

### Step 1 — Duplicate Check

1. Read `wiki/sources-index.md` (one file read — O(1) lookup).
2. Read only the first 20 lines of the incoming file or its sidecar (`head: 20`) to scan frontmatter. Do not read the full file yet.
3. Check for a `project:` frontmatter field:
   - **`project:` field present** → stop, load `AGENTS-PROJECTS.md`, and follow the project ingest workflow. Do not process as a general wiki source.
4. Check for a `source:` frontmatter field containing a URL:
   - **No `source:` URL present** → leave the file in `raw/` untouched. Append a `blocked` entry to today's log file explaining that the `source:` URL is missing and manual intervention is required. Stop.
   - **`source:` URL present** → continue to sub-step 5.
5. Check whether the incoming file's URL appears in `wiki/sources-index.md`:
   - **URL found** → true duplicate. Move the incoming file to `raw/skipped/`. Append a `skipped` entry to today's log file. Stop.
   - **URL not found** → no duplicate. Check whether a matching filename exists in `wiki/sources/` — if so, log a `warning` about the filename collision. Continue to Phase 2.

---

## Phase 2 — Ingest (full source read and content work)

1. Read the full source file. For non-markdown files with a sidecar: use an appropriate reading strategy for the file type — extract text from PDFs, describe images, etc. The sidecar body may also contain free-text notes to incorporate.
2. Discuss key takeaways with the user if helpful.
3. Write a summary page in `wiki/` (e.g., `wiki/sources/article-title.md`). Include wikilinks to all related entity and concept pages — these wikilinks are the cross-reference candidate list for Step 5.
4. Update `wiki/index.md` with the new page.

---

## Phase 3 — Commit (make the ingest durable before the expensive fan-out)

5. Append an entry to today's log file (`wiki/logs/YYYY-MM-DD.md`): `## N. ingest | Source Title`
6. Move the source file from `raw/` to `raw/processed/`. For non-markdown files, move the sidecar alongside it. Update the `source:` frontmatter field in the wiki summary page to reflect the new path (`raw/processed/filename.ext`).
7. Append a new entry to `wiki/sources-index.md`.

---

## Phase 4 — Cross-referencing (expensive fan-out; session-loss safe after Phase 3)

8. Read all entity and concept pages linked from the summary page written in Step 3 in a single batched file read. Update each one.
9. Cross-link those updated pages back to the summary page.
10. If the source is a YouTube video clipped with the Obsidian Web Clipper: fetch the channel name from the YouTube source URL and add `channel_name: Channel Name` to the summary page frontmatter. (Deferred to here to avoid blocking Phase 3 on a network call.)

A single source may touch many wiki pages. That is expected and correct.
