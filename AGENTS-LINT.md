# Lint Module

This module is loaded when the user sends a message beginning with `lint:fast` or `lint:deep`. Follow the matching workflow below exactly.

---

## `lint:fast` — Structural health check (no page reads)

Reads only meta-index files. Completes in ~5 tool calls. Use this for routine checks.

1. Read `wiki/wiki-stats.md` — check section counts and last lint date.
2. Read `wiki/logs/index.md` — check recent activity.
3. Read `wiki/project-status.md` — check project count and last updated.
4. Read `wiki/lint-manifest.md` — scan for:
   - Pages with 0 inbound links (orphans)
   - Pages with unresolved outbound links (missing targets)
   - Source pages with `Author entity? = no` (missing entity pages)
   - Pages not updated since last lint (stale candidates)
5. Report findings. Flag any suspects for `lint:deep` if warranted.
6. Append a structured log entry using the `lint:fast` format in `AGENTS-FORMAT.md`:
   - Keep the heading compact: `## N. lint:fast | Short verdict`
   - Include `Scope`, `Status`, `Issues found`, `Changes made`, and `Verdict` sections.
   - If counts are corrected, include an `Actual counts` line and identify each correction under `Issues found`.
   - Record when no changes were made as `**Changes made:**` followed by `- None.`

Do **not** open any page files during `lint:fast`.

---

## `lint:deep` — Full content audit (targeted page reads)

Use when `lint:fast` surfaces suspects, or when the user explicitly requests a deep pass.

1. Run steps 1–4 of `lint:fast` first to build the suspect list.
2. Read only the flagged pages (orphans, broken links, stale candidates) in a single batched read — not all pages.
3. For each suspect:
   - Look for contradictions with other pages.
   - Find stale claims superseded by newer sources.
   - Find concepts mentioned but lacking their own page.
   - Verify `author:` fields: path-explicit wikilink, entity page exists, entity backlinks to this source.
4. Suggest new questions to investigate and new sources to look for.
5. Update `wiki/lint-manifest.md` with any corrections.
6. Update `wiki/wiki-stats.md` with current counts.
7. Append a log entry: `## N. lint:deep | Summary`
