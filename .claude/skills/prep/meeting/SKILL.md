---
name: prep-meeting
description: |
  Pre-populates a meeting note with context from vault, Slack, and Notion.
  Output is ready for live capture before the meeting starts. Triggers:
  "prep this meeting", "create meeting note with context", "/prep meeting".
model: claude-sonnet-4-6
argument-hint: "[Meeting Name] | [Description] | [optional context]"
allowed-tools: >
  Read Write Glob Grep
  mcp__claude_ai_Slack__slack_search_public
  mcp__claude_ai_Slack__slack_search_public_and_private
  mcp__claude_ai_Notion__notion-search
  mcp__claude_ai_Notion__notion-fetch
---

# /prep:meeting — Meeting prep with context

Gather context from vault, Slack, and Notion. Produce a pre-populated meeting note ready for live capture.

## Input

`$ARGUMENTS` = pipe-delimited string: `[Meeting Name] | [Description] | [optional context]`.

Parse on `|`:
- `meeting_name` — the title (e.g., "Quarterly Roadmap Review")
- `description` — what it's about (e.g., "discuss Q3 priorities with Sam and Priya")
- `extra_context` (optional) — attendees, topics, stakes, urgency

If no `|`, treat the entire string as `description` and derive `meeting_name` from it.

## Instructions

### Step 1 — Read context

1. Load `vault-config` for `personTiers`, `directReports`, `stakeholders`, `projects`, `notion`, `vault.layout`.
2. Load `obsidian-markdown` for syntax.
3. Load `meeting-notes` for the structure and templates.

### Step 2 — Identify attendees and entities

From the description and extra context extract:
- **Attendees** — named people, strip `@`, match against `<vault.layout.people>/@First Last.md`. Cross-reference `personTiers` for importance.
- **Projects/initiatives** — match against `<vault.layout.projects>/` and `vault-config.projects`.
- **Type** — recurring (named series) or one-off?

### Step 3 — Gather vault context

1. **Person notes:** for each attendee, read their note if it exists.
2. **Prior meetings:** check `<vault.layout.meetings>/[meeting_name]/` for previous instances. If none, search `<vault.layout.meetings>/` for notes mentioning the same attendees or topics. Read the most recent 1–2.
3. **Prior 1:1s:** if the meeting is with a single person or small group, also check `<vault.layout.oneOnOnes>/`.
4. **Project notes:** for any referenced projects, read the relevant note.
5. **Open action items:** grep for incomplete tasks mentioning attendees or topic.

### Step 4 — Gather external context

1. **Slack:** `slack_search_public` for recent messages on the topic or by attendees. Look for active threads, recent messages from key attendees, blockers/decisions/updates.
2. **Notion:** `notion-search` with `content_search_mode: "workspace_search"` for pages on the topic. Fetch the most relevant 1–2 with `notion-fetch`. Prioritize pages from `vault-config.notion` if they match.

If either tool is unavailable, skip and note the gap.

### Step 5 — Create the meeting note

**Path:** `<vault.layout.meetings>/[meeting_name]/YYYY-MM-DD.md`. Create the subfolder if missing.

If a file exists at that path, read and ask before overwriting.

Build using the structure from `meeting-notes` skill, with:

#### Frontmatter
```yaml
---
date: YYYY-MM-DD
tags:
  - meeting
  - [contextual tags from topic]
status: draft
attendees:
  - "[[@Person1]]"
  - "[[@Person2]]"
absent:
related:
  - "[[Project or Note]]"
---
```

#### Header
Fill known values; leave blanks for the genuinely unknown.

#### Context section
Pre-populate `> [!note] Why This Matters` with 2–3 sentences:
- What decision/update/outcome this meeting drives
- How it connects to the user's current priorities (from `vault-config.projects` and recent vault activity)
- Any urgency or stakes

Add 1–2 sentences below the callout if research surfaced relevant background.

#### Discussion Topics
Pre-populate 2–5 proposed agenda items based on what's known. Each topic:
- Specific, action-oriented heading
- 1–2 sentences below explaining why it's on the agenda

Sequence by urgency. If a topic has direct evidence, note the source briefly (e.g., *From [[2026-02-17]]* or *Per Slack thread in #channel*).

Leave `### Topic N` placeholders if fewer than 2 known.

#### Decisions, Action Items, Observations
Empty template stubs — filled during/after the meeting.

#### Raw Notes
Leave the collapsed callout stub.

### Step 6 — Output summary

- Confirm file path
- List pre-populated discussion topics
- List identified attendees with person-note status (found / not found)
- Note any gaps (missing Slack context, Notion unavailable, no prior meeting notes)
- Flag any open action items from prior meetings worth reviewing before this one

## Notes

- Wikilink everyone: `[[@First Last]]` for people, `[[Project]]` for projects.
- `related:` should link to the most relevant vault notes from research.
- `status: draft` always — user changes it after the meeting.
- Don't over-fill: prepared starting point, not a pre-written report. Topics get context, not conclusions.
- Tone: write for the user — framing should help them walk into the room knowing what matters and why.
