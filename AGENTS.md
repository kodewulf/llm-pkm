# Wiki Schema

This is the schema file for the Personal Knowledge Base wiki. It tells you how the wiki is structured, what the conventions are, and what workflows to follow.

**Vault root:** `.` (repository root)

All file paths in this schema and its modules are relative to the vault root unless stated otherwise.

---

## Directory Structure

```
llm-pkm/
├── AGENTS.md                ← this file (schema / configuration)
├── AGENTS-FORMAT.md         ← canonical formats for all meta-index and log files
├── AGENTS-INGEST.md         ← ingest workflow
├── AGENTS-LINT.md           ← lint workflows
├── AGENTS-JOURNAL.md        ← journal module
├── AGENTS-PROJECTS.md       ← project module
├── AGENTS-TIL.md            ← TIL module
├── AGENTS-KB.md             ← KB module
├── raw/                     ← immutable source documents (read, never modify)
│   ├── processed/           ← sources that have been successfully ingested
│   └── skipped/             ← sources rejected as true duplicates
└── wiki/                    ← LLM-generated markdown files (you write and maintain)
    ├── index.md             ← content-oriented catalog of all wiki pages
    ├── project-status.md    ← live summary of all project statuses
    ├── wiki-stats.md        ← lightweight meta-index of wiki activity and counts
    ├── sources-index.md     ← machine-readable URL → summary page lookup table (duplicate detection)
    ├── lint-manifest.md     ← per-page link graph + author entity status (used by lint:fast)
    ├── logs/                ← append-only daily log files
    │   ├── index.md         ← reverse-chronological list of log days with daily summaries
    │   └── YYYY-MM-DD.md   ← one file per day; entries use HH:MM:SS timestamps
    ├── til/                 ← dated personal discovery entries (TIL module)
    └── kb/                  ← evergreen knowledge base articles (KB module)
```

### Sidecar Files

Non-markdown files (PDFs, images, etc.) cannot carry frontmatter. To ingest them, drop a sidecar metadata file alongside the source file in `raw/`. The sidecar uses the source filename with `.meta.md` appended:

```
raw/
  my-paper.pdf
  my-paper.pdf.meta.md     ← sidecar for my-paper.pdf
  screenshot.png
  screenshot.png.meta.md   ← sidecar for screenshot.png
```

The sidecar is a standard markdown file with YAML frontmatter. It carries all the fields that would normally appear in the source file's frontmatter:

```yaml
---
source: https://arxiv.org/abs/xxxx
title: My Paper Title
author: Author Name
project:
tags: []
---

Optional free-text notes or context about this file.
```

Required fields: `source` (URL). Optional: `title`, `author`, `project`, `tags`, and any free-text notes in the body.

The sidecar is the identity document for the binary file. The agent reads the sidecar for routing and duplicate detection, then reads the binary file for content.

---

## Layers

**raw/** — The source of truth. Articles, papers, images, data files dropped here by the user. Treat every file in `raw/` as immutable. Never modify, move, or delete anything here. Sidecar files (`*.meta.md`) are also immutable — move them alongside their binary file when moving to `raw/processed/` or `raw/skipped/`.

**wiki/** — Everything here is LLM-generated and LLM-maintained. You create pages, update them when new sources arrive, maintain cross-references, and keep everything consistent. The user reads it; you write it.

**AGENTS.md** — This file. The configuration that makes you a disciplined wiki maintainer rather than a generic chatbot.

---

## Wiki Page Conventions

- All wiki pages are Markdown (`.md`) files inside `wiki/`.
- Use Obsidian-style wikilinks for cross-references: `[[Page Name]]`.
- Keep filenames lowercase with hyphens (e.g., `transformer-architecture.md`).
- Every page should have a brief one-line description at the top (used in `index.md`).
- Pages fall into rough categories: **source summaries**, **entity pages**, **concept pages**, **comparisons**, **overview/synthesis**, **TIL entries**, **KB articles**. Label them consistently.

### Source Author Links

For source summary pages, the `author:` frontmatter field is part of the wiki graph and must be maintained.

- If the author is a meaningful person, organisation, publication, channel, or project, `author:` should use a wikilink to an entity page.
- Prefer path-explicit links with a clean display alias: `author: "[[entities/matt-wolfe|Matt Wolfe]]"`.
- Every linked author entity should exist under `wiki/entities/`.
- The corresponding entity page should link back to the source through `sources:` frontmatter and/or a `Referenced In` section.
- If an author is unknown or not meaningful enough to track as an entity, leave `author:` blank rather than inventing an entity.

---

## Module Routing

If the user's message starts with a recognised prefix, follow this procedure exactly:

1. **Stop.** Do not guess at the workflow or begin writing files.
2. **Read the module file** at the path shown in the table below. Paths are relative to the repository root.
3. **Follow the instructions in that file exactly.** Do not proceed from memory or approximation.

| Prefix | Module file | Purpose |
|---|---|---|
| `ingest:` | `AGENTS-INGEST.md` | Ingest a source file into the wiki |
| `lint:fast` | `AGENTS-LINT.md` | Structural health check (no page reads) |
| `lint:deep` | `AGENTS-LINT.md` | Full content audit (targeted page reads) |
| `journal:` | `AGENTS-JOURNAL.md` | Journal entries — grounded responses saved as dated files |
| `project:` | `AGENTS-PROJECTS.md` | Project research, source tracking, and guide generation |
| `til:` | `AGENTS-TIL.md` | Personal discoveries and fixes, saved as dated TIL entries |
| `kb:` | `AGENTS-KB.md` | Evergreen knowledge base articles, optionally built from source URLs |

**Natural language fallback:** If the user asks to process or ingest a file without using the `ingest:` prefix, treat it as an `ingest:` trigger — read `AGENTS-INGEST.md` and follow it.

**Automated ingest:** If a file in `raw/` has a `project:` frontmatter field, read `AGENTS-PROJECTS.md` and follow the project ingest workflow instead of the standard ingest workflow. This redirect happens inside the ingest flow, not at the routing table level.

**Format reference:** If you need to create or repair any meta-index or log file, read `AGENTS-FORMAT.md` for the canonical format.

---

## Query

When the user asks a question (no prefix):
1. Read `wiki/index.md` to find relevant pages.
2. Read those pages.
3. Synthesize an answer with citations to wiki pages or raw sources.
4. If the answer is valuable (a good comparison, analysis, or discovery), consider filing it as a new wiki page and updating the index and log.

---

## General Principles

- The wiki is a persistent, compounding artifact. Cross-references should already exist. Contradictions should already be flagged. The synthesis should already reflect everything ingested so far.
- You own the wiki layer entirely. The user reads it; you write and maintain it.
- Good answers to queries can be filed back as new wiki pages. Explorations should compound, not disappear into chat history.
- Raw sources are immutable. The wiki is the layer you work in.
- When in doubt about structure, keep it simple. The right structure will emerge from the content.
