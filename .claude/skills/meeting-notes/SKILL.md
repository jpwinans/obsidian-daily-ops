---
name: meeting-notes
description: |
  Encapsulates meeting note format conventions and the algorithm for
  restructuring raw notes/transcripts into the standard template.
  Bundles meeting/1on1/recurring templates. Auto-loaded by
  /meeting:extract, /prep:meeting, /prep:1on1, /prep:day.
user-invocable: false
model: inherit
allowed-tools: Read
---

# meeting-notes — meeting note format and extraction logic

You are the canonical reference for how meeting notes look in this vault and how to convert raw notes/transcripts into that structure.

## When to use this skill

Other skills load you when they need to create or restructure a meeting note. They will say "use the meeting-notes skill format" or pass you a raw note path to restructure.

Bundled templates live alongside this `SKILL.md`:
- `templates/meeting.md` — generic meeting (ad-hoc or recurring)
- `templates/1on1.md` — 1:1 with a single person
- `templates/recurring.md` — recurring meeting series with running context

## Standard meeting note structure

### Frontmatter (required on every note)

```yaml
---
date: YYYY-MM-DD
tags:
  - meeting
  - [additional contextual tags]
status: draft
attendees:
  - "[[@Person Name]]"
related:
  - "[[Project or Note]]"
---
```

For 1:1s use `tags: [- 1-1]` and add a `person:` field.

### Body structure

```markdown
# Meeting Title — YYYY-MM-DD

**Date:** YYYY-MM-DD HH:MM
**Attendees:** [[@Person1]], [[@Person2]], ...
**Absent:** [if known]
**Type:** Recurring / Ad-hoc — description
**Facilitator:** [if known]

---

## Context

> [!note] Why This Matters
> Brief framing of why this meeting matters and what it connects to.

[1-2 paragraphs of context]

---

## Discussion Topics

### Topic 1 Title

[Structured notes with key points, callouts for asides]

> [!info] Related
> Cross-references to vault notes or external links.

### Topic 2 Title

---

## Decisions

| Decision | Owner |
|----------|-------|
| Decision description #decision | [[@Owner]] |

---

## Action Items

Items assigned to the user — use Tasks plugin format with checkboxes:

- [ ] Action item description ⏫ 📅 YYYY-MM-DD
- [ ] Another item @context 🔼 📅 YYYY-MM-DD

## Action Items (Others — For Awareness)

Tracked for context but not the user's to-dos:

- [[@Person]] — Action they own
- [[@Person]] — Another action

---

## My Observations

> [!tip] Relevance
> How this connects to the user's role, priorities, and current initiatives.

### Observations

1. **Observation title:** What this means for the user
2. **Another observation:** Strategic implication or follow-up

---

## Raw Notes

> [!faq]- Notes (click to expand)
> Original raw notes preserved here. Do NOT paste raw transcripts — extract structured content above and discard the transcript or link to it externally.
```

## Extraction algorithm (raw notes → structured)

Given a raw note or transcript:

### 1. Read context
Read `CLAUDE.md` (via `vault-config`) for conventions. Read `obsidian-markdown` for syntax reference.

### 2. Research vault context
Search the vault for any people, projects, or initiatives mentioned:
- `People/` for person notes (use `[[@Name]]` format)
- `Efforts/Projects/` for project references
- `Atlas/` for relevant knowledge notes
- Recent `Calendar/Meetings/` and `Calendar/1-1s/` for related prior discussions

### 3. Restructure
- Preserve existing frontmatter; add fields, never remove
- Convert raw content into the structure above
- Backlink every person with `[[@Name]]`
- Backlink every project with `[[Project Name]]`
- Use callouts (`> [!note]`, `> [!info]`, `> [!warning]`) for important asides
- Use highlight syntax (`==text==`) for critical metrics or statements
- Tag decisions inline with `#decision`
- Only create checkboxes (`- [ ]`) for items assigned to the user (per `vault-config.userIdentity` if provided)
- List others' items without checkboxes under "Action Items (Others)"
- Add priority emoji (⏫ high, 🔼 medium) and due dates (`📅 YYYY-MM-DD`)
- Add `@context` tags: `@meeting`, `@review`, `@blocked`, `@delegate`, `@discuss`

### 4. Critical rule — never paste raw transcripts
Do not paste raw meeting transcripts into the note, even inside a collapsed callout. Extract the structured content into "Discussion Topics", "Decisions", "Action Items", and "Observations". If a transcript exists, link to it externally (Notion page, Drive doc) or delete it. Pasting transcripts pollutes search and review.

### 5. Output
Edit the file at the provided path. Summarize what was extracted:
- Number of decisions captured
- Number of action items (user's vs. others)
- Key observations
- New wikilinks created (people or projects that may need their own notes)

## Templates

When creating a new meeting note from scratch (not extracting), copy the appropriate template from `templates/` in this skill directory and fill in the placeholders. The calling skill tells you whether to use `meeting.md`, `1on1.md`, or `recurring.md`.
