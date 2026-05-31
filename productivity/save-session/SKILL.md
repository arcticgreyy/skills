---
name: save-session
description: >
  Saves a structured summary of the current conversation to a knowledge base
  as a searchable markdown note. ALWAYS use this skill when the user says
  anything like: "save session", "save this chat", "save this conversation",
  "log this", "log what we did", "write up what we did", "make a note of what
  we decided", "add this to my knowledge base", "summarize this session",
  "write a summary of this conversation", "i want to be able to find this later",
  "capture this", "record what we talked about", or any variation suggesting
  they want to preserve this chat for future reference. Also trigger when the
  user says "we're done" or "that's it for today" after a substantial work
  session — proactively offer to save it. The note extracts decisions (with
  reasoning), actions taken, facts learned, and open items — not a transcript,
  but a curated record that future AI sessions can search and use as context.
---

# Save Session

The goal is to distill this conversation into a durable, searchable note —
not a transcript, but a curated record of what actually matters. Future AI
sessions should be able to read this note and instantly understand what was
decided, built, or learned here without needing the full chat history.

## Configuration

Set these paths based on your knowledge base structure:
- **Sessions directory**: `~/knowledge-base/_meta/sessions/` (or your preferred location)
- **Reindex command** (optional): Path to your knowledge base indexer, if available

## What to extract

Think about what a smart colleague would write down after this meeting:

- **Decisions** — choices made and the reasoning behind them (not just what,
  but why — this is the most valuable part)
- **Actions taken** — concrete things that were installed, created, configured,
  or changed, with relevant paths/commands
- **Facts learned** — discoveries, findings, or clarifications that weren't
  known at the start
- **Open items** — anything unresolved, deferred, or flagged as a next step
- **Context** — a one-paragraph summary of what this session was about, written
  so someone with no prior context can orient quickly

Leave out: greetings, clarifying back-and-forth, tool output noise, anything
that's captured better in the actual files created.

## Output format

Generate the note content, derive a short topic slug from the main subject
(e.g., `mcp-setup`, `career-docs-import`, `project-auth-refactor`), then save
and index it.

### Filename
`~/knowledge-base/_meta/sessions/YYYY-MM-DD-{topic-slug}.md`

Use today's actual date.

### Note structure

```markdown
---
title: {Short descriptive title}
type: note
permalink: knowledge-base/meta/sessions/{YYYY-MM-DD-topic-slug}
---

# {Short descriptive title}

**Date**: YYYY-MM-DD
**Duration**: {short | medium | long session}
**Topic**: {one-line summary}

---

## Context

{1–2 paragraph orientation. What problem or goal drove this session?
What was the starting state?}

## Decisions

- **{Decision}**: {Why this choice was made over alternatives, if relevant}
- ...

## Actions Taken

- {Concrete action} — {result or artifact, e.g. file path, command used}
- ...

## Facts & Findings

- {Something learned or confirmed that's worth remembering}
- ...

## Open Items

- [ ] {Next step or unresolved question}
- ...
```

Omit any section that has nothing to put in it — a session that's purely
informational might have no "Actions Taken", for example.

## Steps

1. Read through the full conversation and draft the note content mentally
2. Determine today's date and the topic slug
3. Write the file to `~/knowledge-base/_meta/sessions/YYYY-MM-DD-{slug}.md`
4. (Optional) Run your knowledge base reindex command if configured:
   Example: `your-indexer reindex --project knowledge-base`
   (Skip this step silently if no indexer is configured — the note is still saved.)
5. Confirm to the user: tell them the filename and give a one-line summary
   of what was saved. Keep it brief — they don't need the note read back to them.
