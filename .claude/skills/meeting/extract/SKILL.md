---
name: meeting-extract
description: |
  Transforms a raw meeting note or transcript into the standard
  structured note format (frontmatter, context, discussion topics,
  decisions, action items, observations). Backlinks people and projects.
  Edits the file in place. Triggers: "extract this meeting note",
  "structure my raw notes", "/meeting extract <path>".
model: claude-sonnet-4-6
argument-hint: "[path/to/raw-note.md]"
allowed-tools: Read Write Edit Glob Grep
---

# /meeting:extract — Restructure raw notes

Transform a raw meeting note or transcript into a structured, actionable note matching the standard quality pattern.

## Input

`$ARGUMENTS` = file path to a note containing raw meeting notes or transcript.

## Instructions

### Step 1 — Read context

1. Load `vault-config` for `vault.layout`, `personTiers`, `projects`, `conventions`.
2. Load `obsidian-markdown` for syntax.
3. Load `meeting-notes` for the canonical meeting note structure and the extraction algorithm.
4. Read the raw note at the provided path.

### Step 2 — Research context

Search the vault for context on people, projects, or initiatives mentioned:

- `<vault.layout.people>/` for person notes (`[[@Name]]` format)
- `<vault.layout.projects>/` for project references
- `Atlas/` for relevant knowledge notes
- Recent `<vault.layout.meetings>/` and `<vault.layout.oneOnOnes>/` for related prior discussions

### Step 3 — Restructure per the meeting-notes skill

Apply the structure documented in `meeting-notes/SKILL.md`. Frontmatter:

```yaml
---
date: YYYY-MM-DD
tags:
  - meeting
  - [contextual tags]
status: active
attendees:
  - "[[@Person Name]]"
related:
  - "[[Project or Note]]"
---
```

Body sections (per `meeting-notes`):
- Context (with `> [!note] Why This Matters` callout)
- Discussion Topics (synthesized, not transcript-quoted)
- Decisions (table with owners)
- Action Items (only checkboxes for the user's tasks; others under "Action Items (Others)")
- My Observations (1–3 strategic observations)
- Raw Notes (collapsed callout — DO NOT paste raw transcripts; link externally if needed)

### Step 4 — Apply quality standards

- Backlink every person with `[[@Name]]`
- Backlink every project with `[[Project Name]]`
- Use callouts (`> [!note]`, `> [!info]`, `> [!warning]`) for important asides
- Use highlight syntax (`==text==`) for critical metrics or statements
- Tag decisions inline with `#decision`
- Only create checkboxes (`- [ ]`) for items assigned to the user
- List others' items without checkboxes under "Action Items (Others)"
- Add priority emoji (⏫ high, 🔼 medium) and due dates (`📅 YYYY-MM-DD`)
- Add `@context` tags: `@meeting`, `@review`, `@blocked`, `@delegate`, `@discuss`
- Preserve existing frontmatter — add fields, never remove

### Step 5 — Write the structured note

Edit the file at the provided path with the restructured content. Print a summary:
- Number of decisions captured
- Number of action items (yours vs. others)
- Key observations
- New wikilinks created (people or projects that may need their own notes)

## Notes

- **Never paste raw transcripts** — not inline, not in collapsed callouts. Synthesize.
- Apply the user's tier mapping for ownership decisions on action items.
- If the raw note already has structure, keep what's working and tighten what isn't.
