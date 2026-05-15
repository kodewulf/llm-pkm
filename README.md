# LLM-PKM

An LLM-maintained personal knowledge manager. Raw sources go in; structured wiki pages, evergreen articles, TIL entries, journal responses, and project guides come out — all cross-linked and kept consistent by an AI agent.

---

## How It Works

Drop source files into `raw/` (or paste a URL), then ask the agent to process them. The agent writes and maintains everything under `wiki/`. You read; the agent writes.

Prefix your message to route to a specific module:

| Prefix | Module | What it does |
|--------|--------|--------------|
| *(none)* | Core wiki | Ingest, query, or lint the wiki |
| `til:` | TIL | Record a personal discovery from this conversation |
| `kb:` | Knowledge Base | Create or update an evergreen reference article |
| `journal:` | Journal | Get a grounded response, saved as a dated entry |
| `project:` | Projects | Research a topic and compile a guide |

---

## Installation

### Prerequisites

LLM-PKM works with any AI agent runtime that auto-loads project context files from the workspace directory. It has been tested with:

- **Claude Desktop** — set the project folder as your workspace
- **Claude Code** — launch from the project root
- **OpenAI Codex** — launch from the project root
- **OpenCode** — launch from the project root
- **Hermes** (CLI/TUI or via Open WebUI) — launch from the project root

The common pattern: the tool reads `AGENTS.md` (or `CLAUDE.md`) from the project root on startup. That file turns a generic agent into a wiki maintainer. No additional configuration is needed for CLI/TUI agents — just `cd` into the folder and start the tool.

Desktop applications (Claude Desktop, Open WebUI) require you to set the project folder as the workspace/root directory in the application settings.

### Setup

1. Clone this repository:
   ```
   git clone <repo-url> <path-to-your-pkm>
   cd <path-to-your-pkm>
   ```

2. Open the folder in your chosen agent runtime (see Prerequisites above).

3. Start a conversation. The agent will load `AGENTS.md` and follow the wiki schema automatically. On first run, it will create the initial index files (`wiki/index.md`, `wiki/sources-index.md`, `wiki/logs/index.md`, `journal/index.md`, `projects/index.md`) — no manual setup required.

### Claude Desktop Users

Claude Desktop does not always reliably auto-load `CLAUDE.md` when working in the project folder. To ensure the wiki schema is always active, add the following to your **Project Instructions** in Claude Desktop settings:

```
At the start of every conversation, read the file at:
<path-to-your-pkm>/CLAUDE.md

Follow the instructions found there. This file points to AGENTS.md in the same
directory, which contains the full wiki schema and operational instructions.
Load AGENTS.md and follow it for all operations in this project.
```

Replace `<path-to-your-pkm>` with the actual path where you cloned the repository (e.g. `~/pkm` or `D:\projects\ai\llm-pkm`).

---

## Directory Structure

```
llm-pkm/
├── AGENTS.md               ← core schema and operational instructions
├── AGENTS-FORMAT.md        ← canonical formats for meta-index and log files
├── AGENTS-INGEST.md        ← ingest workflow
├── AGENTS-LINT.md          ← lint workflows
├── AGENTS-JOURNAL.md       ← journal module
├── AGENTS-KB.md            ← KB module
├── AGENTS-PROJECTS.md      ← projects module
├── AGENTS-TIL.md           ← TIL module
├── CLAUDE.md               ← points to AGENTS.md (Claude-specific entrypoint)
│
├── raw/                    ← immutable source documents (never modified)
│   ├── assets/             ← non-markdown assets (images, PDFs, etc.)
│   ├── processed/          ← sources successfully ingested
│   └── skipped/            ← true duplicates, rejected at ingest
│
├── wiki/                   ← LLM-generated and LLM-maintained
│   ├── concepts/           ← concept pages
│   ├── entities/           ← entity pages (people, tools, products)
│   ├── kb/                 ← evergreen KB articles
│   ├── logs/               ← append-only daily log files
│   ├── sources/            ← one summary page per ingested source
│   └── til/                ← dated TIL entries
│
├── journal/                ← personal journal entries + index
└── projects/               ← project research folders + index
```

---

## Modules

### Ingest

> **Trigger:** no prefix — natural language or drop a file into `raw/`

Processes a file from `raw/` into the wiki. Files without a `source:` URL are blocked and logged; true duplicates are moved to `raw/skipped/`.

**Workflow:**

1. Checks `wiki/sources-index.md` for duplicates (requires a `source:` URL in frontmatter).
2. Writes a summary page under `wiki/sources/`.
3. Updates `wiki/index.md`, `wiki/log.md`, and `wiki/sources-index.md`.
4. Moves the file to `raw/processed/`.
5. Updates all linked entity and concept pages.

**File format:** frontmatter (`source`, `title`, `author`, `tags`) + summary, key points, and entity links.

---

### Query

> **Trigger:** no prefix — ask a question

Answers questions by reading `wiki/index.md`, pulling relevant pages, and synthesising a cited answer. Valuable answers can be filed as new wiki pages.

**Workflow:**

1. Read `wiki/index.md` to find relevant pages.
2. Read those pages.
3. Synthesise an answer with citations to wiki pages or raw sources.
4. If the answer is valuable (a good comparison, analysis, or discovery), consider filing it as a new wiki page and updating the index and log.

---

### Lint (`lint:`)

> **Trigger:** `lint: <fast|deep>` or ask to lint or health-check the wiki

Health-checks the wiki: finds contradictions, orphan pages, stale claims, missing cross-references, and concepts without pages.

**Two forms:**

- `lint:fast` — structural health check with no page reads.
- `lint:deep` — full content audit with targeted page reads.

---

### TIL (`til:`)

> **Trigger:** `til: <description of what you learned>`

Records a personal discovery directly from conversation — no external source URL required.

**Workflow:**

- Synthesises the problem, what was tried, and the resolution from the conversation.
- Writes a dated file to `wiki/til/YYYY-MM-DD-slug.md`.
- Updates `wiki/index.md` and `wiki/log.md`.
- No confirmation step — written immediately.

**File format:** frontmatter (`date`, `learned_by`, `tags`, `related`) + sections: Context, What Was Tried, Resolution, Notes.

---

### Knowledge Base (`kb:`)

> **Trigger:** `kb: <topic>` or `kb: <article-name> <url>`

Creates or updates evergreen reference articles that grow over time.

**Two forms:**

- `kb: <topic>` — synthesises from existing wiki content.
- `kb: <article-name> <url>` — fetches a URL and synthesises from it plus wiki context.

**Workflow:** draft → user confirmation → write. On confirmation:

- Writes or updates `wiki/kb/article-name.md` (merges into existing articles rather than replacing).
- Updates `wiki/index.md` and `wiki/log.md`.
- For URL-sourced articles, registers the URL in `wiki/sources-index.md`.

**File format:** frontmatter (`created`, `updated`, `tags`, `sources`, `related`) + sections: Overview, Key Concepts, Common Issues/Gotchas, Examples, References.

---

### Journal (`journal:`)

> **Trigger:** `journal: <your entry>`

Provides a grounded, wiki-informed response to a personal entry and saves it. Links go one way: journal entries reference wiki pages, never the reverse.

**Workflow:**

1. Reads relevant wiki pages and past journal entries.
2. Responds with specific advice citing wiki content and patterns from past entries.
3. Flags recurring themes when the same topic appears three or more times.
4. Saves the entry (original + response + synthesis + related links) to `journal/YYYY-MM-DD-title.md`.
5. Updates `journal/index.md` and `wiki/log.md`.

**File format:** original entry + response + synthesis + related wiki links, saved to `journal/YYYY-MM-DD-title.md`.

---

### Projects (`project:`)

> **Trigger:** `project: <project-name>` or `project: <project-name> <url>`

Manages multi-source research projects with a compiled guide as the deliverable.

**Two forms:**

- Tag a clipped file with `project: project-name` in frontmatter — detected automatically on ingest.
- `project: project-name <url>` — fetch and ingest a URL immediately.

**Operations** (detected from message intent):

| Intent | Trigger phrase | Action |
|--------|---------------|--------|
| New project | Project name not in index | Scaffold folder and files |
| Ingest source | URL or tagged raw file | Add to sources, update notes |
| Query | Question | Search project + wiki, synthesise |
| Generate guide | "generate guide" / "build guide" | Compile `guide.md` |
| Status | "status" / "update" | Summarise current state |

Each project lives at `projects/project-name/` with four files: `overview.md`, `sources.md`, `notes.md`, `guide.md`. Notes compound on every ingest — new sources are integrated into the running synthesis, not appended in isolation. The guide is opinionated: it states best approaches, flags contradictions, and is written to be followed directly.

**File format:** four files per project — `overview.md`, `sources.md`, `notes.md`, `guide.md` — under `projects/project-name/`.

---

## Key Files

| File | Purpose |
|------|---------|
| `wiki/index.md` | Catalog of all wiki pages by category — start here for queries |
| `wiki/log.md` | Append-only chronological record of every operation |
| `wiki/sources-index.md` | URL → wiki page table; used for duplicate detection on every ingest |
| `journal/index.md` | Reverse-chronological index of all journal entries |
| `projects/index.md` | Catalog of all projects with status |

---

## Conventions

- All wiki filenames: lowercase with hyphens (`transformer-architecture.md`).
- Cross-references: Obsidian wikilinks (`[[Page Name]]`).
- Every wiki page has a one-line description at the top (used in `wiki/index.md`).
- `wiki/log.md` and `wiki/sources-index.md` are append-only — existing rows are never edited or deleted.
- `raw/` is immutable — files are never modified, only moved to `raw/processed/` or `raw/skipped/`.

---

## Obsidian

LLM-PKM uses Obsidian-style wikilinks (`[[Page Name]]`) for cross-references throughout the wiki. To browse and visualise the knowledge graph, open the project folder as an Obsidian vault:

1. Install Obsidian: <https://obsidian.md/download>
2. Open the project folder as a vault: **File → Open folder as vault** → select the `llm-pkm` directory
3. Obsidian will immediately render all wikilinks, backlinks, and the graph view

The wiki is plain Markdown — Obsidian is the viewer, the AI agent is the writer.
