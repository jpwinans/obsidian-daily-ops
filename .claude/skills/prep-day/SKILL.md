---
name: prep-day
description: |
  Reads Google Calendar for a target day, classifies each event (1-1 /
  recurring meeting / ad-hoc / ignore), then runs the appropriate prep
  skill for every non-ignored event. Rewrites the "Today's Meetings"
  dashboard with the day's shape. Triggers: "prep my day", "prep today's
  meetings", "/prep day".
model: claude-sonnet-4-6
argument-hint: "[YYYY-MM-DD] (defaults to today)"
allowed-tools: >
  Read Write Edit Glob Grep
  mcp__claude_ai_Google_Calendar__list_calendars
  mcp__claude_ai_Google_Calendar__list_events
---

# /prep:day — Daily meeting prep orchestrator

Read Google Calendar for the target day, classify each event, then run the appropriate prep skill for each non-ignored meeting.

## Input

`$ARGUMENTS` = optional `YYYY-MM-DD`. Defaults to today.

## Instructions

### Step 1 — Read context

1. Load `vault-config` for `personTiers`, `projects`, `directReports`, `stakeholders`, `conventions`, `vault.layout`. Note `conventions.meetingIgnorePatterns` (default: `Lunch, Focus Time, Block, OOO`).
2. Determine target date (`$ARGUMENTS` or today).

### Step 2 — Fetch calendar events

Use `list_calendars` to identify the user's primary work calendar (the one matching their email domain or labeled "primary").

Use `list_events` for the target date in the user's local timezone. If the user's TZ isn't obvious from the calendar metadata, default to the system locale.

### Step 3 — Classify each event

Apply exactly one classification per event:

#### `ignore`
Skip if **any** is true:
- All-day event
- Title contains any pattern in `conventions.meetingIgnorePatterns`
- Title contains: `OOO`, `Out of Office`, `Holiday`, `No Meeting`
- No other attendees (user is the only person) and the title contains no person names
- User's RSVP status is `declined`
- Duration < 15 minutes (likely a reminder)

#### `1-1`
- Title contains `1:1`, `1-1`, `1 on 1`, `one on one` (case-insensitive), OR
- Title matches `[Person] <> [User]`, `[User] <> [Person]`, or `[Person] / [User]`, OR
- Exactly 2 attendees (user + one other person)

If 1-1, extract the other person's name. Strip email domain, title prefix, or `@` to get a clean name.

#### `meeting` (recurring)
- Has a `recurringEventId`, AND
- Not classified as 1-1

#### `ad-hoc`
- No `recurringEventId`, AND
- Not 1-1 or ignore

### Step 4 — Match to vault

For each non-ignored meeting, find its vault folder name:
1. Exact match: `<vault.layout.meetings>/[Event Title]/`
2. Fuzzy match: list `<vault.layout.meetings>/` and find the closest subfolder (ignore date suffixes, punctuation, abbreviations)
3. No match: use the calendar event title — it may be new

That folder name (or title if no folder) is `meeting_name`.

### Step 5 — Show classification table

Print before running any preps:

```
## Today's Meetings — [Date]

| Time | Event | Classification | Prep |
|------|-------|---------------|------|
| 9:00 AM | [Title] | Meeting (recurring) | prep:meeting "[Title]" |
| 10:30 AM | 1:1 [Person] | 1-1 | prep:1on1 "[Person]" |
| 2:00 PM | [Title] | Ad Hoc | prep:meeting "[Title] | as Ad Hoc" |
| 3:00 PM | Focus Time | **ignore** | — |
```

Then: "Running prep for [N] meetings. Starting now..."

Do not pause for confirmation — proceed.

### Step 6 — Run prep for each non-ignored meeting

In chronological order:

- **1-1** → invoke `prep:1on1` with the other person's name
- **Recurring meeting** → invoke `prep:meeting` with the vault folder name (or event title)
- **Ad hoc** → invoke `prep:meeting` with `[event title] | as Ad Hoc`

**Sub-skill failure handling:** if a `prep:1on1` or `prep:meeting` invocation fails (MCP unavailable, write permission denied, malformed input), capture the error and continue with the next meeting. Do not abort the whole orchestration. The Step 8 final summary records each meeting's status (`✓ created`, `✓ updated`, or `✗ failed: <reason>`). One bad prep should not derail the rest of the day.

### Step 7 — Rewrite the Today's Meetings dashboard

**Only when target date is today.** Skip for past or future dates.

Completely overwrite `<vault root>/📆 Today's Meetings.md` with:

```markdown
---
date: YYYY-MM-DD
tags:
  - dashboard
  - meetings
status: active
---
# 📆 Today's Meetings — [Weekday], [Month] [D], [YYYY]

> [!info] Day Shape
> [N] prepped meetings ([M] ignored: [comma-separated ignored titles]). Critical path: **[one-sentence framing of the day's most important thread, drawn from the prep notes you just wrote and today's daily note]**.

## Schedule

| Time | Meeting | Type | Prep Note |
|------|---------|------|-----------|
| [time] | [event] | [Recurring / 1-1 / Ad Hoc / *ignored*] | [[Calendar/.../[meeting_name]/YYYY-MM-DD\|→ Prep]] or — |

## Headlines per Meeting

### [time] — [meeting name]
[1–3 sentence headline drawn from the prep note's Context callout. Bold the most consequential phrase. End with → [[full path to prep note]].]

---

## Today's ⏫ Forcing Functions

These cross-cut multiple meetings — surface in whichever room makes sense first.

- [ ] **[Task]** — [one-line context] [tags] [priority] 📅 YYYY-MM-DD

---

*Generated by `/prep:day` for [[<vault.layout.daily>/YYYY-Mon/YYYY-MM-DD|[Weekday MM/DD]]]. Replace tomorrow morning by re-running.*
```

Rules:
- Schedule table is chronological. Ignored events get `*ignored*` and `—`. Use Obsidian wikilinks with display text (escape `|` as `\|`).
- Headlines: one short paragraph per non-ignored meeting. Pull the headline from each prep note's "Why This Matters" callout. **Bold one specific phrase**.
- Forcing Functions: 5–8 items max. Pull from today's daily note's Top 3 + Risk Digest plus cross-cutting items in prep notes' carry-forwards. Tasks-plugin syntax.
- Day Shape: 1–2 sentences. Help the user walk into their first meeting knowing what the day is, not just what's on it.

After writing, print: `✓ Dashboard rewritten: 📆 Today's Meetings.md`.

### Step 8 — Final summary

```
## Prep Complete

| Meeting | Status | Note Path |
|---------|--------|-----------|
| ... | ✓ created | <vault.layout.meetings>/.../YYYY-MM-DD.md |
```

List any meetings where prep was skipped or failed, with the reason.

## Notes

- If no non-ignored meetings: say so and exit cleanly. Do not write or rewrite the dashboard.
- If Calendar MCP is unavailable: say so and exit. Do not guess at meetings.
- If the calendar returns zero events for the day (genuinely empty calendar): report "No meetings scheduled for [date]" and exit without writing the dashboard.
- `prep:meeting` and `prep:1on1` run their full research and note creation flows — no need to pre-gather data here.
- If a prep note already exists for today, the individual prep skill will ask before overwriting.
- For 1-1s, cross-reference the person's name against `<vault.layout.people>/`. If no person note exists, note it but still run `prep:1on1`.
