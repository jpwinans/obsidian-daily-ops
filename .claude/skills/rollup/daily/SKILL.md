---
name: rollup-daily
description: |
  End-of-day rollup. Reads what actually happened (1:1s, meetings,
  modified notes, completed tasks) and reconciles the daily note in
  place — every section reflects end-of-day reality, not just an
  appended End-of-Day block. Triggers: "rollup my day", "close out
  today", "/rollup daily".
model: claude-sonnet-4-6
argument-hint: "[YYYY-MM-DD or 'yesterday'] (defaults to today)"
allowed-tools: Read Write Edit Glob Grep
---

# /rollup:daily — End-of-day reconciliation

Transform the daily note from a morning plan into a complete record of the day. Update every section with what actually happened.

## Input

`$ARGUMENTS` = optional date. `YYYY-MM-DD`, natural language ("yesterday", "Monday"), or empty (today).

## Instructions

### Step 1 — Read context

1. Load `vault-config` for `vault.layout`, `conventions`.
2. Load `obsidian-markdown` for syntax.
3. Read the Daily Note template at `<vault.layout.templates>/Daily Note.md`.

### Step 2 — Determine target date

Resolve `$ARGUMENTS` to `YYYY-MM-DD`. Empty → today.

### Step 3 — Run meeting transcript ingest (optional)

If a `meeting:ingest` skill is installed (it ships with this suite), invoke it with a 1-day lookback (or wider for past dates) so vault meeting notes are enriched with transcript content before reading them.

Wait for ingest to complete. If it finds no new transcripts, note that briefly and continue.

### Step 4 — Gather the day's actual activity

Find all notes created or modified on the target date. This is the source of truth.

1. **1:1 notes:** read files matching `<vault.layout.oneOnOnes>/*/YYYY-MM-DD.md`
2. **Meeting notes:** read files in `<vault.layout.meetings>/` with the target date (check `date` frontmatter too)
3. **Modified project notes:** check `<vault.layout.projects>/` for files modified on the target date
4. **Modified person notes:** check `<vault.layout.people>/` for files modified on the target date
5. **Action Items file:** read `Action Items.md` at vault root if it exists — check for items with the target date
6. **Other modified notes:** check `Atlas/` and `Efforts/` for any files modified on the target date

### Step 5 — Read the existing daily note

Path: `<vault.layout.daily>/YYYY-Mon/YYYY-MM-DD.md`. Create the month subfolder if needed.

- Missing → create from template, fill in date.
- Present → read completely; you'll update in place.

### Step 6 — Reconcile plans vs. reality

Walk every section of the existing daily note and compare planned vs. actual.

For each item:
- **Completed:** evidence in a meeting note, 1:1, modified file, or marked `[x]`
- **Partially done:** started but not finished
- **Not done:** no evidence — carry forward
- **Moot:** no longer relevant
- **New items:** things that happened but weren't in the morning plan

### Step 7 — Update the daily note in place

**CRITICAL:** Update the entire note. The end-of-day note should read as a coherent record, not a morning plan with an appendix.

#### Morning Briefing — annotate in place

Update Top 3 and Overnight Alerts with end-of-day status. Add inline annotations:

```markdown
> [!tip] Top 3 Items Needing Attention
> 1. **1:1 with [Person]** — ✅ Met. [Outcome] → [[Calendar/1-1s/[Person]/YYYY-MM-DD]]
> 2. **[Item]** — ✅ Attended. [Outcome] → [[link]]
> 3. **[Item]** — ❌ Deferred → 📅 YYYY-MM-DD
```

The reader should see at a glance: planned → outcome.

#### Tasks — update every line

For each task (Overdue, Due Today, etc.):
- **Completed:** check off (`- [x]`), add `✅` with brief note or link
- **Deferred:** keep `- [ ]`, append `→ deferred 📅 NEW-DATE` with brief reason
- **Moot:** strike through or note why
- **Partially done:** note what happened and what remains
- **Unchanged:** leave as-is

```markdown
**Due Today (8 items — 3 completed, 4 deferred, 1 moot):**
- [x] Task ✅ → [[link]]
- [ ] Task — ❌ deferred → 📅 YYYY-MM-DD
- [ ] ~~Task~~ — moot, [reason]
```

Include tallies in section headers so throughput is visible at a glance.

#### Focus — annotate outcome

```markdown
## Focus

> What is the ONE thing that matters most today?

- [Stated focus] — ✅ Achieved / ❌ Not started, [reason]
```

#### Meetings — replace placeholders with actuals

Replace `### 00:00 - Meeting 1` placeholders. For each:
- One-line summary of the key outcome
- Link to the full note
- Don't duplicate meeting content

#### Day Log — preserve and extend

Keep any Capture/Day Log entries the user wrote. Add non-meeting activity from Step 4 (Slack threads, document reviews, vault maintenance).

#### Stale Relationships / Inbox — update if changed

If a relationship was contacted today, note it. If inbox items were processed, update the count.

#### End of Day — structured close-out

```markdown
## End of Day

### Tasks Completed Today
- [x] Task description ⏫ → [[source]]

### Carried Forward
- [ ] Unfinished task — brief reason ⏫ 📅 YYYY-MM-DD
- [ ] New task surfaced during the day 📅 YYYY-MM-DD

### Key Emerging Themes
1. **Theme** — One-line insight connecting multiple meetings or observations
2. **Theme** — Another pattern

**Energy:** N/5
**Key decision made:** [or "None — observation day"]
**Carried forward:** [1-line summary of what moves to tomorrow]
```

#### General revision rules

1. **Every section gets touched.** No section frozen in morning state while End of Day tells a different story. If unchanged, add a brief note ("Unchanged from morning").
2. **Tasks live in ONE place.** Don't list a task in Morning Briefing AND Carried Forward AND End of Day. Update where it first appears, then reference in End of Day.
3. **Link, don't duplicate.** Meeting summaries = 1–2 lines max with `[[wikilink]]`. The daily note is an index.
4. **Preserve Capture entries.** Keep what the user wrote during the day; add to it.
5. **Tallies make scanning fast.** Use counts in section headers.
6. **New tasks go in Carried Forward**, not injected into the morning's task lists.

### Step 8 — Stdout summary

- Number of meetings/1:1s logged
- Tasks completed vs. carried forward vs. moot
- Key themes identified
- Gaps (meetings without notes, planned items with no evidence either way)
- Delta from morning: what changed most from the plan

## Notes

- The daily note is a living document — by EOD every section should reflect reality.
- Carry forward, don't delete. Unfinished tasks move to Carried Forward with updated due dates.
- Wikilinks for vault refs. People as `[[@Person]]`.
- Daily notes go in `<vault.layout.daily>/YYYY-Mon/`, never at vault root.
- If the daily note already has an End of Day section with content, merge intelligently — don't create duplicates.
