---
name: action-items-scan
description: |
  Reads the user's central action-items file and presents open tasks
  bucketed by age (stale / aging / due today / due this week / horizon).
  Updates the file's scan-timestamp header. Output to stdout. Triggers:
  "scan action items", "what's stale", "/action-items scan".
model: claude-haiku-4-5
allowed-tools: Read Edit Glob Grep
---

# /action-items:scan — Open task triage by age

Read `📋 Action Items.md`, present open tasks organized by urgency. Oldest and most overdue first, so the user can orient quickly.

## Input

No arguments. Uses today's date.

## Instructions

### Step 1 — Read context + action-items file

1. Load `vault-config` for `vault.layout` (root path) and `conventions`. Apply the documented `vault-config` contract.
2. Read `📋 Action Items.md` at vault root. Expected structure:
   - `## 🎯 This Week` — manually curated weekly focus (≤5 items)
   - `## Open` with `### Strategic`, `### Operational`, `### Relational`
   - `## Waiting On Others` — pending on other people, with `⚡ escalate by:` sub-lines
   - `## Someday / Maybe` — intentionally deferred items with review-by dates
   - `## Completed` — ignore

If the file doesn't exist or has no open tasks, say so clearly and stop.

### Step 2 — Parse sections

- `## 🎯 This Week` → extract for the "This Week" output block (show as-is, no re-bucketing)
- `## Open / ### Strategic|Operational|Relational` → combine into the Open pool for age-bucketing; preserve the work-type label per item
- `## Waiting On Others` → tasks + their `⚡ escalate by:` sub-lines
- `## Someday / Maybe` → **do not scan**. Count them but don't surface in age buckets

For each Open task, extract:
- Full task text (strip `- [ ] ` prefix and `→ [[...]]` source links)
- Work type: `Strategic` / `Operational` / `Relational`
- Priority emoji: ⏫ / 🔼 / *(none)*
- Tags: `#tag` values
- Context: `@context` values
- Due date: `📅 YYYY-MM-DD` (may be absent)

For Waiting On tasks, also extract `⚡ escalate by:` and check if past.

### Step 3 — Bucket by age

`days_overdue = today - due_date` (positive = overdue, negative = upcoming). No due date = "On the Horizon".

| Bucket | Threshold | Label |
|--------|-----------|-------|
| 🔴 Stale | Overdue > 3 days | Stale — Over 3 Days |
| 🟠 Aging | Overdue 1–3 days | Aging — 1–3 Days Overdue |
| 🟡 Due Today | Due today | Due Today |
| 🟢 Coming Up | Due in 1–7 days | Due This Week |
| ℹ️ On the Horizon | Due in 8–30 days OR no date | On the Horizon |

Within each bucket sort: ⏫ first, then 🔼, then no emoji. Within same priority, by due date (earliest first), then alphabetically.

**Note:** `## 🎯 This Week` items are shown in their own block at top — do NOT re-bucket them, even if dates are overdue.

### Step 4 — Update file header

Touch only these fields in `📋 Action Items.md`:

1. Frontmatter `date:` → today
2. H1 heading `# Action Items — YYYY-MM-DD` → today
3. Info callout: append/update `Last scanned: YYYY-MM-DD.`

Do not modify any tasks, sections, or other content.

### Step 5 — Generate output to stdout

```markdown
# Action Items Scan — YYYY-MM-DD HH:MM

## 🎯 This Week
*(Curated weekly focus — N items)*

- **[Task]** ⏫ #tag — *due YYYY-MM-DD*

---

## 🔴 Stale — Over 3 Days Overdue

> [N tasks] These have been sitting longest. If they haven't moved, something is wrong.

- **[Task]** ⏫ `Strategic` #tag — *N days overdue* (📅 YYYY-MM-DD)

---

## 🟠 Aging — 1–3 Days Overdue

> [N tasks] Getting stale. Needs attention before they compound.

[items...]

---

## 🟡 Due Today

> [N tasks]

[items...]

---

## 🟢 Due This Week

> [N tasks] Due in the next 7 days. Plan for these.

[items...]

---

## ℹ️ On the Horizon

> [N tasks] Due in 8–30 days or no date set. Awareness only.

[items...]

---

## Waiting On Others

> [N items] — Cannot action directly. ⚡ = escalation trigger.

- **[[@Person]]:** [Task] — *due YYYY-MM-DD* ⚡ escalate by YYYY-MM-DD
- **[[@Person]]:** [Task] — *no date* ⚡ **OVERDUE** escalate by YYYY-MM-DD ← flag if past

---

*Scan completed at YYYY-MM-DD HH:MM. N open tasks (N This Week + N Strategic + N Operational + N Relational). N parked in Someday / Maybe. N waiting on others. Top 3 most urgent: [list]*
```

### Formatting rules

- **Task text:** trim source links and repetitive context. Truncate at ~120 chars with `…` if long.
- **Age label:** always relative (`*5 days overdue*`, `*due in 3 days*`) — never just the date.
- **Priority inline:** ⏫ or 🔼 inline after task text, before work type label and tags.
- **Work type label:** `Strategic` / `Operational` / `Relational` in backtick code style.
- **Tags:** show all `#tag` values for quick scanning.
- **@context:** show `@blocked` prominently — these may need unblocking before moving.
- **Escalation triggers:** in Waiting On, always show `⚡ escalate by:`. If today or past, flag with `⚡ **OVERDUE**`.
- **Empty sections:** omit entirely.

## Notes

- Output to stdout. The only file write is the header update in `📋 Action Items.md`.
- 30-second orientation, not a deep review — keep lines short and scannable.
- This Week items are never re-bucketed — respect the user's curation.
- Someday / Maybe items are never shown — count in footer only.
- Tasks with `@blocked` are special: note them explicitly even in lower age buckets.
- Waiting On is always last regardless of due dates — these aren't the user's actions.
- **Ownership rule:** when a task references a Slack message or meeting note, verify who it's actually directed at. A request from a manager to a group is not automatically the user's action item unless they're explicitly named or the topic falls in their domain. Tasks where someone else is clearly the owner go to "Waiting On Others" or are omitted — never in the user's Open pool.
