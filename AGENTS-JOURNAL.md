# Journal Module

This file is loaded when the user's message starts with `journal:`. It defines how journal entries are handled — how they are responded to, saved, and indexed.

---

## Directory Structure

```
journal/
├── index.md                    ← catalog of all entries (date, title, one-line summary)
└── YYYY-MM-DD-entry-title.md   ← one file per journal entry
```

---

## On a `journal:` Message

When the user sends a message prefixed with `journal:`, follow these steps in order:

1. **Strip the prefix** — treat everything after `journal:` as the journal entry content.
2. **Read `wiki/index.md`** — identify wiki pages relevant to the topics raised in the entry.
3. **Read relevant wiki pages** — load concept, entity, or source pages that relate to what the user is journaling about.
4. **Read `journal/index.md`** — check for past entries on the same or related topics.
5. **Read relevant past entries** — if past entries touch the same theme, load them for additional context.
6. **Compose a response** grounded in:
   - Content from the wiki (cite specific pages where relevant)
   - Patterns from past journal entries (note recurring themes if present)
   - Your own knowledge and reasoning
   Provide helpful advice, insights, guidance, and ideas. Be direct and specific — not generic.
7. **Save the entry** — write a new file to `journal/` with the filename format `YYYY-MM-DD-entry-title.md`. Choose a short, descriptive title based on the content. The file should contain:
   - The user's original entry
   - Your full response
   - A short synthesis section summarising key takeaways
   - Related wiki pages (as wikilinks)
8. **Update `journal/index.md`** — append a new row with the date, title (as a link to the entry file), and a one-line summary.
9. **Append to `wiki/logs/YYYY-MM-DD.md`** — format: `## N. journal | Entry title — one-line summary`

---

## Journal Entry File Format

```markdown
---
type: journal-entry
date: YYYY-MM-DD
title: "Short Descriptive Title"
tags:
  - journal
---

# Short Descriptive Title

## Entry

<user's original text>

## Response

<your response, grounded in wiki and past entries>

## Synthesis

<2-4 sentence summary of key takeaways and any actions or questions to carry forward>

## Related

- [[concepts/relevant-concept]]
- [[sources/relevant-source]]
- [[journal/YYYY-MM-DD-related-entry]] (if applicable)
```

---

## journal/index.md Format

```markdown
# Journal Index

| Date | Entry | Summary |
|------|-------|---------|
| [[journal/YYYY-MM-DD-entry-title\|YYYY-MM-DD]] | [[journal/YYYY-MM-DD-entry-title\|Title]] | One-line summary |
```

Entries are listed in reverse chronological order (newest first).

---

## Recurring Pattern Detection

If the same theme, struggle, or topic appears across three or more journal entries, note it explicitly in the response:

> "This is the third time you've journaled about X. The pattern suggests..."

Reference the earlier entries by wikilink so the user can navigate back to them.

---

## Cross-Linking Rules

- Journal responses should cite wiki pages where relevant using wikilinks: `[[concepts/llm-wiki-pattern]]`
- Journal entries should **not** be listed in `wiki/index.md` — they live in `journal/index.md`
- Wiki pages should **not** be updated to reference journal entries — the link goes one way (journal → wiki)
- Past journal entries may reference each other freely

---

## General Principles

- Responses are grounded in the wiki first — do not answer from a blank slate when relevant wiki content exists.
- Be specific. If the wiki has a concept page that directly addresses what the user is journaling about, reference it explicitly and draw from it.
- The journal is personal. Tone should be warm, direct, and conversational — not clinical.
- Every entry is saved. Nothing disappears into chat history.
