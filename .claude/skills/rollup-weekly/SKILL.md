---
name: rollup-weekly
description: |
  Weekly review generator. Aggregates a completed week's daily notes,
  meetings, and 1:1s into a structured retrospective with accomplishments,
  in-progress, blockers, decisions, incomplete tasks, and next-week focus.
  Triggers: "weekly review", "rollup last week", "/rollup weekly".
model: claude-opus-4-7
effort: high
argument-hint: "[YYYY-Www or YYYY-MM-DD] (defaults to last completed week)"
allowed-tools: Read Write Glob Grep
---

# /rollup:weekly — Weekly retrospective

Aggregate the week's activity into a structured weekly review.

## Input

`$ARGUMENTS` = optional week identifier. `YYYY-Www`, a date, or empty. Empty → previous completed week (last Mon–Fri).

## Timing

**This skill is always a retrospective of a completed week.** Never run mid-week on the current week. If `$ARGUMENTS` matches the current week, warn and ask whether to proceed or wait.

## Instructions

### Step 1 — Read context

1. Load `vault-config` for `vault.layout`, `projects`, `conventions`. Apply the documented `vault-config` contract.
2. Load `obsidian-markdown` for syntax.
3. Read the Weekly Note template at `<vault.layout.templates>/Weekly Note.md`. If missing, fall back to minimal weekly-note frontmatter (`date`, `tags: [weekly]`, `week: YYYY-Www`, `status: draft`) and continue.

### Step 2 — Determine week range

Compute the Monday–Friday range. `$ARGUMENTS` parsing:
- `YYYY-Www` → parse directly
- A date → find the week containing it
- Empty → previous completed week

### Step 3 — Gather the week's data

Read all relevant notes from the week:

1. **Daily notes:** all files in the relevant `<vault.layout.daily>/YYYY-Mon/` subfolder(s) within the date range. If the week spans two months, check both.
2. **Meeting notes:** files in `<vault.layout.meetings>/` (including subdirs) with dates in range.
3. **1:1 notes:** files matching `<vault.layout.oneOnOnes>/*/YYYY-MM-DD.md` within range.
4. **Project status:** read active project notes from `<vault.layout.projects>/`.
5. **OKR status:** read `<vault.layout.outcomes>/Current OKRs.md` if it exists.

**If zero source notes are found across all of the above** (fresh clone, vacation week, vault used only sporadically), stop here and report: "No vault activity in the week of [Mon date] – [Fri date]. Nothing to roll up. Run `/morning:start` and `/rollup:daily` during the week to populate daily notes that this skill can aggregate." Do not write an empty weekly review.

### Step 4 — Extract and categorize

From the gathered notes:

- **Accomplishments:** completed tasks (`- [x]`), decisions made (`#decision`), milestones hit
- **In Progress:** open tasks grouped by project/initiative
- **Blockers:** items tagged `@blocked` or mentioning blockers
- **Decisions Made:** all `#decision` items with context
- **People interactions:** who was met with, key conversations

### Step 5 — Generate the weekly review

Write to `<vault.layout.weekly>/YYYY-Www.md`:

```yaml
---
date: YYYY-MM-DD
tags:
  - weekly
week: YYYY-Www
status: draft
---
```

```markdown
# Week of Month D, YYYY

## Accomplishments

- [Completed items extracted from daily notes and meeting decisions]

## In Progress

### [[Project Name]]
- [Open items related to this project]

### [[Another Project]]
- [Open items]

## Blockers

- [Items tagged @blocked or identified as stuck]

## Decisions Made This Week

| Decision | Date | Context |
|----------|------|---------|
| Decision description | YYYY-MM-DD | [[Source Meeting Note]] |

## Incomplete Tasks

```tasks
not done
due before YYYY-MM-DD
group by due
```

## Next Week Focus

1. [Priority based on upcoming deadlines and carried items]
2. [Follow-up from this week's decisions]
3. [Upcoming meetings or milestones]

## People to Follow Up With

[List anyone overdue for contact based on `last-contact` and `contact-frequency` frontmatter]

## Reflection

**What went well:**
**What to improve:**
**What am I avoiding:**
```

### Step 6 — Present for review

Display:
- Number of accomplishments captured
- Number of open items carried
- Number of decisions logged
- Any blockers identified

Then: "Draft written to `<vault.layout.weekly>/YYYY-Www.md` with status: draft. Adjust anything before marking it active?"

## Notes

- **Always retrospective.** Past tense for accomplishments. All daily notes for the week should exist — note any gaps but don't treat the week as incomplete.
- Wikilinks for all vault refs. People as `[[@Person]]`.
- `status: draft` initially — let the user mark it `active` after review.
- If daily notes don't exist for some days, note the gap.
- Group in-progress items by project for scanability.
