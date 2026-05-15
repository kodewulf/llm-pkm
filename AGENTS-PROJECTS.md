# Projects Module

This file is loaded when the user's message starts with `project:`, or when a file in `raw/` contains a `project:` frontmatter field. It defines how projects are created, researched, and compiled into guides.

---

## Directory Structure

```
projects/
├── index.md                  ← catalog of all projects (name, status, one-line summary)
└── project-name/
    ├── overview.md           ← goal, status, open questions, wiki cross-links
    ├── sources.md            ← all sources ingested for this project
    ├── notes.md              ← running research notes and synthesised findings
    └── guide.md              ← generated implementation guide (the deliverable)
```

Project folder names are lowercase with hyphens, derived from the project name (e.g. `my-test-project`).

---

## Source Input Methods

### 1. Web Clipper tag (primary — automated)
Add a `project:` field to the frontmatter of any file clipped with the Obsidian Web Clipper:

```yaml
---
title: "How to Build a RAG Pipeline"
source: "https://..."
created: 2026-05-09
tags:
  - clippings
project: my-test-project
---
```

The hourly cron job (or a manual ingest trigger) will detect the `project:` field and route the file to the correct project automatically. No additional action needed.

### 2. Link paste (escape hatch — immediate)
Start a message with `project:` followed by the project name and a URL:

```
project: my-test-project https://some-article.com
```

The agent will fetch the URL, extract the content, and ingest it into the project immediately.

---

## Operations

### Detect Intent

When a `project:` message arrives, determine intent from the content:

| Intent | Trigger | Action |
|---|---|---|
| New project | Project name not in `projects/index.md` | Scaffold project folder and files |
| Ingest source | URL or tagged raw file present | Run project ingest workflow |
| Query | Question about a project | Search project files + wiki, synthesise answer |
| Generate guide | "generate guide", "build guide", "compile guide" | Run guide generation workflow |
| Status / update | "update", "status", "progress" | Summarise current state of the project |

---

### New Project

When a project name is not found in `projects/index.md`:

1. Create the folder `projects/project-name/`.
2. Create `overview.md` with the project name, goal (ask the user if not provided), status `active`, and empty sections for open questions and wiki cross-links.
3. Create empty `sources.md`, `notes.md`, and `guide.md` with placeholder headings.
4. Add the project to `projects/index.md`.
5. Append to today's log file (`wiki/logs/YYYY-MM-DD.md`): `## N. project | Created project: Project Name`

---

### Project Ingest (from raw/ tag or link paste)

When a source is being added to a project:

1. Read the source content (from `raw/` file or fetched URL).
2. Extract key findings, methods, steps, and any caveats.
3. Append a new entry to `projects/project-name/sources.md`:
   - Title, source URL, date, and a concise summary (5–10 bullet points).
4. Update `projects/project-name/notes.md`:
   - Integrate the new findings into the running synthesis.
   - Note any contradictions with previous sources.
   - Note any gaps the new source fills or opens.
5. Update `projects/project-name/overview.md`:
   - Increment source count.
   - Update open questions if the source answers or raises any.
   - Add relevant wiki cross-links if the source relates to existing wiki concepts.
6. If the file came from `raw/`, update the `source:` field in the raw file's frontmatter to `raw/processed/filename.md` and move it to `raw/processed/`.
7. Append a new row to `wiki/sources-index.md` with the source URL and project file path (e.g. `projects/local-ai/sources#source-slug`). This registers the URL for duplicate detection across both general wiki ingests and project ingests.
   - Project-ingest rows intentionally point to `projects/.../sources#...` anchors rather than `wiki/sources/...` summary pages. Project sources are scoped to their project and should not be mirrored into the general wiki source-summary layer unless the user explicitly asks to promote them.
8. Append to today's log file (`wiki/logs/YYYY-MM-DD.md`): `## N. project | Ingested source into Project Name: Source Title`

---

### Query

When the user asks a question about a project:

1. Read `projects/project-name/overview.md` and `notes.md`.
2. Read `projects/index.md` if the project name is ambiguous.
3. Cross-reference `wiki/index.md` for any relevant concept or entity pages.
4. Synthesise an answer with citations to project files and wiki pages.
5. If the answer is valuable, consider appending it to `notes.md` as a named finding.

---

### Generate Guide

When the user asks to generate or update the guide for a project:

1. Read `projects/project-name/sources.md` — all ingested sources and their summaries.
2. Read `projects/project-name/notes.md` — synthesised findings and contradictions.
3. Cross-reference `wiki/index.md` for any relevant wiki concept or entity pages and read them.
4. Synthesise all of the above into a structured, step-by-step implementation guide written to `projects/project-name/guide.md`.
5. The guide should:
   - Be opinionated — where sources agree, state the best approach clearly.
   - Flag disagreements — where sources contradict each other, note both approaches and recommend one with reasoning.
   - Be actionable — written so the user can follow it directly without referring back to the sources.
   - Cite sources — reference `sources.md` entries and wiki pages where relevant.
   - Include: Overview, Prerequisites, Step-by-step instructions, Common pitfalls, and Further reading.
6. Update `projects/project-name/overview.md` to note the guide was generated and the date.
7. Append to today's log file (`wiki/logs/YYYY-MM-DD.md`): `## N. project | Generated guide for Project Name`

---

## projects/index.md Format

```markdown
# Projects Index

| Project | Status | Summary |
|---------|--------|---------|
| [[projects/project-name/overview\|Project Name]] | active | One-line summary |
```

Status values: `active`, `paused`, `complete`.

---

## overview.md Format

```markdown
---
type: project-overview
project: "Project Name"
status: active
created: YYYY-MM-DD
sources: 0
---

# Project Name

## Goal

<what this project is trying to achieve>

## Status

active

## Open Questions

- <question raised by research>

## Wiki Cross-Links

- [[concepts/relevant-concept]]
- [[entities/relevant-entity]]

## Notes

- Guide generated: no
- Last source ingested: —
```

---

## sources.md Format

```markdown
# Sources — Project Name

## Source Title
- **URL:** https://...
- **Date ingested:** YYYY-MM-DD
- **Summary:**
  - Key point one
  - Key point two
  - Key point three
```

---

## notes.md Format

```markdown
# Research Notes — Project Name

Running synthesis of all ingested sources. Updated on every ingest.

## Key Findings

<synthesised findings across all sources>

## Contradictions

<where sources disagree and what the implications are>

## Open Questions

<gaps that remain unanswered>

## Named Findings

<valuable answers from project queries, filed for reference>
```

---

## Cross-Linking Rules

- Project `overview.md` files may link to wiki concept and entity pages freely.
- Wiki pages are **not** updated to reference project files — the link goes one way (projects → wiki).
- Projects may reference each other's `overview.md` if they share relevant research.
- Journal entries may reference project files if relevant.

---

## General Principles

- The guide is the deliverable. Every ingest should make the eventual guide better.
- Notes compound. Each new source is integrated into the existing synthesis, not appended in isolation.
- Be opinionated in the guide. The user is reading it to get a clear recommendation, not another list of options.
- When sources contradict, surface it explicitly — don't silently pick one.
