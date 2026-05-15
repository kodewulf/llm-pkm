# KB Module

This module handles evergreen knowledge base articles — structured reference material that persists and is updated over time. Triggered by the `kb:` prefix.

**Vault root:** `.` (repository root)

All file paths below are relative to the repository root. Use them exactly as written.

---

## Purpose

KB articles are the reference layer of the wiki. Where TIL entries record a moment of learning, KB articles synthesise what is durably known about a topic into a single, updateable document. They are written to be read again, not just filed.

---

## Trigger Forms

### Form 1 — Topic synthesis
```
kb: <topic>
```
Example: `kb: Cron Execution`

Claude queries the wiki (index, concept pages, TIL entries, source summaries) for everything relevant to the topic, synthesises a draft article, and presents it for review before writing.

### Form 2 — URL-sourced article
```
kb: <article-name> <url>
```
Example: `kb: Cron Execution https://man7.org/linux/man-pages/man5/crontab.5.html`

Claude fetches and reads the URL, synthesises a draft article from its content (combined with any relevant wiki context), and presents it for review before writing.

---

## Workflow

Execute all phases in order. Do not write any files until the user confirms the draft.

### Phase 1 — Draft

#### Form 1 (topic synthesis)
1. Read `wiki/index.md` to identify relevant pages (concepts, entities, sources, TIL entries).
2. Read those pages in a single batched read.
3. Synthesise a draft article from the gathered context.
4. Present the draft to the user with a brief note on what sources were drawn from.
5. **Stop. Wait for confirmation before proceeding.**

#### Form 2 (URL source)
1. Read `wiki/sources-index.md` and check whether the URL is already ingested.
   - If already ingested: use the existing summary page as a source rather than re-fetching.
   - If not ingested: fetch and read the URL content.
2. Read any relevant existing wiki pages (same as Form 1 steps 1–2).
3. Synthesise a draft article from the URL content plus wiki context.
4. Present the draft to the user with a brief note on what was used.
5. **Stop. Wait for confirmation before proceeding.**

### Phase 2 — Confirmation

The user responds with one of:
- **`confirm`** (or equivalent affirmative) → proceed to Phase 3 as drafted
- **Corrections or additions** → revise the draft inline, present the updated version, wait again
- **Rejection** → discard, log nothing

### Phase 3 — Write

#### Determine file path
- Derive a kebab-case filename from the article name
- Full file path: `wiki/kb/article-name.md`
- If the file already exists: **update in place** — merge new content into the existing article, preserving any sections the draft doesn't touch
- If the file does not exist: create it

#### Article structure

```markdown
---
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [tag1, tag2]
sources: [[sources/relevant-source]], https://example.com
related: [[concepts/relevant-concept]], [[til/YYYY-MM-DD-slug]]
---

# <Article Title>

> One-sentence summary of the topic.

## Overview

High-level explanation of the topic.

## Key Concepts / Details

The substantive content. Use subheadings as needed.

## Common Issues / Gotchas

Practical pitfalls and how to avoid them (include from TIL entries where relevant).

## Examples

Concrete commands, config snippets, or code (if applicable).

## References

Links to source summaries, TIL entries, or external URLs that informed this article.
```

Rules:
- `sources` may mix wikilinks (for ingested sources) and raw URLs (for Form 2 sources not separately ingested)
- `updated` must be refreshed every time the article is modified
- Omit sections that aren't relevant for the topic
- Keep **Common Issues / Gotchas** populated from TIL entries — this is where personal experience compounds into reference material

#### Update `wiki/index.md`

File: `wiki/index.md`

Read the file first. Add or update the entry under the **Knowledge Base** section:

```
- [[kb/article-name]] — one-line description
```

Write the updated file back. Do not remove or alter any other content.

#### Append to `wiki/log.md`

File: `wiki/logs/YYYY-MM-DD.md`

Append to the end of the file. Do not edit existing content.

For new articles:
```
## N. kb | Created: Article Title
```

For updated articles:
```
## N. kb | Updated: Article Title
```

#### Form 2 only — update `wiki/sources-index.md`

File: `wiki/sources-index.md`

If the URL was freshly fetched (not previously ingested), append a row so it is recognised on future duplicate checks:

```
| <url> | kb/article-name |
```

---

## Query Behaviour

KB articles are listed in `wiki/index.md` under the **Knowledge Base** section and will be pulled into query answers alongside source summaries, concept pages, and TIL entries. They are authoritative — prefer KB article content over raw source summaries when both exist.

---

## Updating Existing Articles

If `kb: <topic>` is triggered and a KB article already exists for that topic:
- Note in the draft presentation that an existing article was found
- Show what is changing or being added
- On confirmation, merge — do not wholesale replace

This allows KB articles to grow incrementally as new TIL entries and sources accumulate.
