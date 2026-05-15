# Format Reference

This file defines the canonical formats for all wiki meta-index and log files. Load it when creating or repairing any of these files.

---

## Lint Manifest (`wiki/lint-manifest.md`)

Machine-readable per-page link graph. Primary input for `lint:fast`; scopes `lint:deep` to suspects only.

```markdown
# Lint Manifest
_Last updated: YYYY-MM-DD_

| Page | Inbound links | Outbound links | Last lint | Author entity? |
|------|--------------|----------------|-----------|----------------|
| sources/article-title | 2 | 5 | YYYY-MM-DD | yes |
| entities/matt-wolfe | 3 | 0 | YYYY-MM-DD | — |
| concepts/rag | 1 | 3 | YYYY-MM-DD | — |
```

**Column rules:**
- **Page** — path relative to `wiki/`, no `.md` extension.
- **Inbound links** — count of other wiki pages that wikilink to this page. `0` = orphan.
- **Outbound links** — count of wikilinks this page makes to other wiki pages. Unresolved targets (no file exists) should be noted as e.g. `3 (1 unresolved)`.
- **Last lint** — date this row was last verified against the actual file.
- **Author entity?** — `yes`, `no`, or `—` (not a source page). `no` = missing entity page or non-path-explicit link.

**Maintenance rules:**
- Add a row for every new page created during ingest (Phase 2, Step 3). Set inbound = 0, outbound = count of wikilinks written, last lint = today, author entity = yes/no/— as appropriate.
- Update inbound counts for all pages linked from a new summary page during Phase 4.
- Update the full manifest after every `lint:deep` pass.
- Do **not** update the manifest on every single-source ingest beyond the new row — counts are approximate between deep lint passes and that is acceptable.

---

## Log Files (`wiki/logs/YYYY-MM-DD.md`)

Each day has its own file named `YYYY-MM-DD.md`. Entries use sequential numbering.

```markdown
# YYYY-MM-DD

## N. operation | Short title

Description body (optional). Time can be noted here if relevant: HH:MM.

---

## N+1. operation | Short title
```

Where `operation` is one of: `ingest`, `query`, `lint:fast`, `lint:deep`, `skipped`, `blocked`, `warning`, `til`, `kb`, `project`, `journal`, `core`, `cron`, `misc`.

- **Sequential numbering** per day. No timestamps on heading lines — they create visual noise and false precision.
- Use `---` separators between entries for visual grouping when descriptions are multi-line.
- Never delete or edit past log entries unless the user explicitly asks for a repair. Normal logging is append-only.
- The title should be short — fits on one line. Put details in the description body.

**To append a log entry:** open only `wiki/logs/YYYY-MM-DD.md` for today. Create the file if it does not exist. Never load other log files for a simple append.

### Operation Selection

- `kb` is only for evergreen knowledge base articles under `wiki/kb/` or actions triggered by the `kb:` workflow.
- `core` is for changes to the PKB operating system itself: `AGENTS.md`, `AGENTS-*.md`, schema/workflow rules, log formats, meta-index formats, migrations, and structural files such as `wiki/project-status.md`, `wiki/wiki-stats.md`, `wiki/lint-manifest.md`, `wiki/sources-index.md`, and `wiki/logs/index.md`.
- `cron` is for automated scheduled checks or maintenance runs.
- `misc` is the fallback for log-worthy work that does not fit any specific operation. Prefer a specific operation whenever one applies.

### Example entries by operation type

```
## N. ingest | Source Title

## N. skipped | filename.md

True duplicate detected. Matches existing source: wiki/sources/filename.md (URL: <url>). Moved to raw/skipped/.

## N. blocked | filename.md

Cannot ingest: no `source:` URL found in frontmatter. File left in raw/. Add a `source:` URL to the frontmatter to allow ingestion.

## N. blocked | filename.pdf

Cannot ingest: no sidecar file found. Create raw/filename.pdf.meta.md with a `source:` URL in frontmatter to allow ingestion.

## N. warning | filename.md

Filename collision: incoming file shares a name with wiki/sources/filename.md but has a different source URL. Proceeding with ingest.
Existing URL: <existing-url>
Incoming URL: <incoming-url>

## N. lint:fast | Short verdict

**Scope:** Read wiki-stats, logs index, project status, and lint manifest only. No page files read.

**Status:**
- PASS — No zero-inbound pages reported in lint-manifest.md.
- PASS — No missing author entities reported in lint-manifest.md.
- WARNING — sources/example-page has unresolved related-page links for entities/example-target.

**Issues found:**
- WARNING — Unresolved outbound links remain in lint-manifest.md:
  - Page: sources/example-page
  - Missing targets: entities/example-target
  - Recommended action: run lint:deep or targeted repair.

**Changes made:**
- None.

**Verdict:** Fast lint found no broad structural failures, but one existing unresolved-link suspect remains.

## N. lint:fast | Manifest stale; wiki-stats corrected

**Scope:** Read wiki-stats, logs index, project status, and lint manifest only. No page files read.

**Actual counts:**
Sources N | Entities N | Concepts N | TIL N | KB N

**Issues found:**
- CRITICAL — lint-manifest.md is missing N pages created since the last lint:deep.
- FIXED — wiki-stats.md had outdated counts; corrected.
- INFO — Existing stale warning remains for sources/example-page.

**Changes made:**
- Updated wiki/wiki-stats.md section counts.
- Did not update wiki/lint-manifest.md; manifest rebuild requires lint:deep.

**Verdict:** Warrants lint:deep to refresh manifest rows and verify link counts.

## N. lint:deep | Summary

## N. query | Query description

## N. til | TIL title

## N. kb | Created: Article Title

## N. kb | Updated: Article Title

## N. core | Updated: AGENTS.md — schema/workflow change

## N. core | Migrated: log structure or meta-index format

## N. cron | Automated ingest check — no new files

## N. misc | Short description

## N. project | Created project: Project Name

## N. project | Ingested source into Project Name: Source Title

## N. project | Generated guide for Project Name

## N. journal | Entry title
```

---

## Log Index (`wiki/logs/index.md`)

```markdown
# Log Index

| Date | Summary |
|------|---------|
| [[logs/2026-05-11\|2026-05-11]] | Batch ingest — 2 sources; lint pass |
| [[logs/2026-05-10\|2026-05-10]] | Batch ingest — 8 sources; TIL: Claude OAuth expiry |
```

Reverse-chronological. One row per day. Append a new row when a new daily file is created. The summary is a single short phrase covering the day's most significant operation — written at the end of the session, not on every entry.

---

## Project Status (`wiki/project-status.md`)

Live summary of all projects. Updated every time a project is created or its status changes.

```markdown
# Project Status
_Last updated: YYYY-MM-DD HH:MM_

## Summary
- Active: N
- Paused: N
- Complete: N

## Projects

| Project | Status | Last Updated | Summary |
|---------|--------|--------------|---------|
| [[projects/project-name/overview\|Project Name]] | active | YYYY-MM-DD | One-line summary |
```

**Maintenance rules:**
- Update whenever a project is created, its status changes, or its one-line summary changes.
- Update simultaneously with the project's `overview.md` — never let them diverge.
- `projects/index.md` and `project-status.md` must always be in sync.

---

## Wiki Stats (`wiki/wiki-stats.md`)

Lightweight meta-index updated after lint passes and significant ingests.

```markdown
# Wiki Stats
_Last updated: YYYY-MM-DD_

## Section Counts

| Section | Files | Last Modified |
|---------|-------|---------------|
| Sources | N | YYYY-MM-DD |
| Entities | N | YYYY-MM-DD |
| Concepts | N | YYYY-MM-DD |
| TIL | N | YYYY-MM-DD |
| KB | N | YYYY-MM-DD |

## Most Recently Updated

- **Source:** [[sources/example-source|Example Source]] — YYYY-MM-DD
- **Entity:** [[entities/example-entity|Example Entity]] — YYYY-MM-DD
- **Concept:** [[concepts/example-concept|Example Concept]] — YYYY-MM-DD

## Recent Log Activity (last 7 days)

| Date | Entries |
|------|---------|
| YYYY-MM-DD | N |
```

**Maintenance rules:**
- Update after every lint pass.
- Update after any batch ingest that creates 3 or more new pages.
- Do not update on every single-source ingest — section counts are approximate between lint passes and that is acceptable.

---

## Sources Index (`wiki/sources-index.md`)

Machine-readable lookup table for duplicate detection only. Maps every ingested source URL to its wiki summary page.

```
| URL | Wiki Page |
|---|---|
| https://example.com/article | sources/article-slug |
```

**Rules:**
- One row per ingested source.
- URL must exactly match the `source:` field in the raw processed file.
- Append a new row at the end of every successful ingest (Phase 3, Step 7).
- Never delete or edit existing rows.
- Do not add skipped or blocked sources — only successfully ingested ones.

---

## Wiki Index (`wiki/index.md`)

Content-oriented catalog of every page in the wiki, organised by category.

```
## Category Name

- [[Page Name]] — one-line description
- [[Another Page]] — one-line description
```

Update on every ingest and whenever new pages are created.
