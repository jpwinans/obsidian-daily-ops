---
name: prep-1on1
description: |
  Generates a 30-minute-ready 1:1 note for a named person — agenda above
  the fold, research below the fold. Pulls from the person note, recent
  1:1s, open action items, Slack mentions, and Notion. Triggers:
  "prep my 1:1", "1-1 prep with [name]", "/prep 1on1 [name]".
model: claude-sonnet-4-6
argument-hint: "[Person Name]"
allowed-tools: >
  Read Write Glob Grep
  mcp__claude_ai_Slack__slack_search_public
  mcp__claude_ai_Slack__slack_search_public_and_private
  mcp__claude_ai_Notion__notion-search
  mcp__claude_ai_Notion__notion-fetch
---

# /prep:1on1 — 1:1 prep for a named person

Gather context and produce a 30-minute-ready 1:1 note: agenda at the top, research below the fold.

## Input

`$ARGUMENTS` = person name (e.g., `Riley Cohen` or `@Riley Cohen` or `People/@Riley Cohen.md`).

Strip `@` prefix, `<vault.layout.people>/` path prefix, and `.md` suffix to get the clean name.

## Instructions

### Step 1 — Read context

1. Load `vault-config` for `personTiers`, `directReports`, `stakeholders`, `projects`, `notion`, `vault.layout`.
2. Load `obsidian-markdown` for syntax.
3. Load `meeting-notes` skill — use its `templates/1on1.md` as the base.

### Step 2 — Gather vault data

1. **Person note:** read `<vault.layout.people>/@[Name].md` if it exists.
2. **Recent 1:1s:** read the last 3 1:1 notes from `<vault.layout.oneOnOnes>/[Name]/`, sorted by date desc.
3. **Open action items:** grep for incomplete tasks mentioning this person.
4. **Recent meeting mentions:** search `<vault.layout.meetings>/` for recent notes mentioning them.

### Step 3 — Gather external data

1. **Slack:** `slack_search_public` for recent messages from or mentioning this person. Read recent activity in their team channels for context on their work.
2. **Notion:** `notion-search` with `content_search_mode: "workspace_search"` for their name. Fetch the most relevant result (e.g., their weekly update page).

If MCPs are unavailable, skip and note.

### Step 4 — Create the 1:1 note

Create `<vault.layout.oneOnOnes>/[Name]/` if needed. Create the note at `<vault.layout.oneOnOnes>/[Name]/YYYY-MM-DD.md`.

If a file exists for today's date, read and ask before overwriting.

**The note is designed for a 30-minute meeting.** Structure for action, not reading. Two zones:

#### Zone 1 — Above the fold (used DURING the meeting)

1. **Agenda** — 3–5 prioritized topics with one-line context each. Mark top 2–3 as ⏫ (must-cover). Add a collapsed "Quick asks" callout for lower-priority items only raised if time permits.
2. **Meeting Notes** — empty section for live capture.
3. **Action Items** — empty.
4. **Wins / Feedback** — empty sections for `[w]` and `[!]` task statuses.

#### Zone 2 — Below the fold (reference, collapsed)

Below a `---` rule, in collapsed `> [!faq]-` callouts:

5. **Open Action Items** — incomplete tasks mentioning this person + the Dataview query.
6. **Context callouts** — one per source: Notion update, Slack highlights, previous 1:1 follow-ups, manager framing (if Tier 2 person flagged this report). Each callout has a descriptive title.
7. **My Updates** — 2–3 bullets about the user's recent relevant activity to share if it comes up.

#### Agenda prioritization

When ordering:

1. **Manager input is highest priority.** Anything flagged by `personTiers` Tier 2 (the user's manager) in recent 1:1s, meetings, or notes goes to the top with ⏫.
2. **Time-sensitive items next.** Deadlines within 48 hours, go/no-go decisions, blockers.
3. **Open action items from previous 1:1s.** Accountability and follow-through.
4. **Signals from Slack/Notion.** Patterns, risks, or opportunities.
5. **Relationship building.** Cadence, working preferences, quick asks — only if early in the relationship or time permits.

No more than 5 numbered items + the quick-asks callout. Excess goes to quick asks or "Notes for Next Time."

### Step 5 — Output summary

- Confirm file path
- List the agenda items
- Note gaps (missing person note, no previous 1:1s, MCP unavailable)

## Notes

- Always create the 1:1 note in `<vault.layout.oneOnOnes>/[Name]/` — primary output.
- If a note already exists for today, read and ask before overwriting.
- If the person note doesn't exist, note it and suggest creating one.
- If no previous 1:1 notes exist, note this may be the first 1:1.
- Wikilink everyone: `[[@Person]]`, `[[Project]]`.
- `related:` frontmatter links to the person's key projects and any referenced notes.
- `status: draft` — user changes to `active` after the meeting.
- Agenda items: enough context to jog memory (one line), NOT full paragraphs. Detail lives in collapsed callouts below.
