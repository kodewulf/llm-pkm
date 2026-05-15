# TIL Module

This module handles personal discovery entries — things learned, problems solved, issues resolved — captured directly from conversation context. Triggered by the `til:` prefix.

**Vault root:** `.` (repository root)

All file paths below are relative to the repository root. Use them exactly as written.

---

## Purpose

TIL entries are dated snapshots of first-person knowledge. They record what was learned and how, so that future queries can surface personal experience alongside external sources. They are not curated reference articles — they are honest records of the moment understanding was reached.

Unlike standard wiki ingest, TIL entries require no external `source:` URL. The conversation itself is the source.

---

## Trigger

Any message beginning with `til:` routes here.

Examples:
- `til: summarise what we've done here and document the resolution`
- `til: document what I learned about cron environment variables`
- `til: we fixed the docker networking issue, save it`

---

## Workflow

Execute all steps in order. Do not skip steps or reorder them.

### Step 1 — Synthesise

Read the current conversation. Extract:
- **What the problem or question was**
- **What was tried or explored**
- **What the resolution or key insight was**
- **Any commands, config snippets, or specific steps taken**

Do not ask the user to summarise. Do the synthesis yourself from the conversation.

### Step 2 — Generate slug and filename

- Derive a short kebab-case slug from the subject (e.g. `cron-env-vars`, `docker-network-bridge-mode`)
- Full file path: `wiki/til/YYYY-MM-DD-slug.md` using today's date

### Step 3 — Write the TIL file

Create the file at the path from Step 2 with the following structure:

```markdown
---
type: til
date: YYYY-MM-DD
title: "Human-readable title"
tags:
  - til
  - tag1
  - tag2
related:
  - "[[concepts/relevant-concept]]"
  - "[[entities/relevant-entity]]"
---

# TIL: <Human-readable title>

> One-sentence summary of what was learned.

## Context

What was being attempted or what problem was encountered.

## What Was Tried

Steps, approaches, or dead ends explored (if relevant).

## Resolution

The fix, insight, or answer. Include specific commands, config, or code if applicable.

## Notes

Any caveats, follow-up questions, or things to investigate further.
```

Rules:
- `tags` must include `til` plus 1–4 lowercase keywords
- `related` is a block list of wikilinks to existing wiki pages; omit the field entirely if nothing fits
- Keep **Resolution** concrete and copy-pasteable where possible — this is what future-you needs
- Omit sections that aren't relevant (e.g. omit **What Was Tried** if the path was direct)

### Step 4 — Update `wiki/index.md`

File: `wiki/index.md`

Read the file first. Add the new entry under the **TIL / Personal Knowledge** section:

```
- [[til/YYYY-MM-DD-slug]] — one-line description of what was learned
```

Write the updated file back. Do not remove or alter any other content.

### Step 5 — Append to `wiki/log.md`

File: `wiki/logs/YYYY-MM-DD.md`

Append to the end of the file:

```
## N. til | TIL title
```

Do not edit any existing content. Append only.

---

## Query Behaviour

When answering a query, Claude reads `wiki/index.md` first. TIL entries are listed there and will be pulled into answers alongside external sources. A TIL entry about cron will surface when the user asks "I'm having trouble with cron, can you help?" — just as a concept page or source summary would.

---

## No Confirmation Required

TIL entries are written immediately — no confirmation step. They are snapshots, not reference articles. If the synthesis is wrong, the user can ask for a correction and the file will be updated.
