---
name: action-items-triage
description: |
  Build an interactive HTML triage page for the consolidated `📋 Action
  Items.md` backlog. Three tabs: Triage (keep / defer / delegate / kill /
  complete decisions), Focus (pick 3-5 rocks for the week), Balance
  (portfolio view across work type, tags, people). Phase 2 of the skill
  applies the decisions JSON back to the vault. Triggers: "triage my
  action items", "daily triage", "what should I work on today", "apply
  decisions", "/action-items triage".
model: claude-sonnet-4-6
allowed-tools: Read Write Bash Glob Grep
---

# /action-items-triage — Interactive prune / focus / balance ritual

Build an interactive triage page that walks through three stages — prune, focus, balance — against the consolidated backlog at `📋 Action Items.md`. Then, after the user finishes in the browser, apply the decisions back to the vault.

## What this skill produces

A single self-contained `.html` file with three tabs:

1. **Triage** — rapid keep / defer / delegate / kill / complete decisions, one task at a time
2. **Focus** — pick 3–5 rocks for `🎯 This Week` from the survivors (cap enforced)
3. **Balance** — portfolio view across work type, category tag, and person attribution

Plus a Phase 2 vault write-back — after the browser session, the page emits a decisions JSON file and the skill applies the changes directly to `📋 Action Items.md`.

## When to trigger

| Phase | User says | What to do |
|-------|-----------|------------|
| Phase 1 (build) | "triage my action items", "daily triage", "what should I work on today" | Parse backlog, score, build HTML, save to vault root |
| Phase 2 (apply) | "apply the decisions", "apply decisions from the JSON", "write back the triage" | Read `action-items-decisions.json`, edit `📋 Action Items.md`, report summary |

---

## Phase 1 — Build the triage page

### 1. Read context + backlog

1. Load `vault-config` for `personTiers`, `vault.layout`. Apply the documented `vault-config` contract. Identify Tier 1-2 people (the user's manager and skip-level) — these drive the BossBoost.
2. Open `📋 Action Items.md` at vault root.
   - **If the file does not exist:** stop with a clear message: "No `📋 Action Items.md` at vault root yet. Run `/action-items-compress` first to consolidate vault tasks into a ranked backlog, then re-run this skill."
   - **If it exists but has no open tasks:** stop and report "Backlog is empty — nothing to triage."
   - Otherwise, parse every open task (`- [ ]`) in the `## Open` section, grouped by Work Type (Strategic / Operational / Relational).
3. Parse `## Waiting On Others` (these get a `waiting: true` flag — shown in triage but default to "Keep" with a different UI treatment) and `## Someday / Maybe` (`someday: true` — only in a collapsed drawer, not surfaced for triage).

Extract for each task:

| Field | How to parse |
|-------|--------------|
| `id` | Stable hash of task text (first 80 chars) — used to match decisions back |
| `text` | Clean task text with emojis/dates/tags/breadcrumbs stripped |
| `raw` | Full original line (preserved for write-back) |
| `workType` | Which `### ` subsection (Strategic / Operational / Relational) |
| `priority` | `high` if `⏫`, `medium` if `🔼`, else `low` |
| `tags` | All `#tag` tokens |
| `dueDate` | ISO date after `📅` (skip `📅 review:` markers — those are Someday) |
| `people` | All `[[@Name]]` wikilinks |
| `sources` | Paths after the `→` arrow |

### 2. Score each task

Compute a `dailyScore` to float time-sensitive work, staleness, unblocking work, and leadership-attributed work to the top.

```
dailyScore = BASE + DeadlineBoost + StalePenalty + BlockerBoost + BossBoost
```

**BASE** — from priority emoji: `⏫` → 800, `🔼` → 500, none → 200.

**DeadlineBoost** — relative to today:
- Overdue: **+500**
- Due today: **+400**
- Due within 7 days: **+200**
- Due within 14 days: **+100**
- No date or >14 days: **+0**

**StalePenalty** — if overdue, add `10 × days_overdue`, capped at **+200**. Amplifies "this has been sliding" over "just due today."

**BlockerBoost** — **+300** if ANY:
- Task contains `#blocker` tag
- Text contains: "unblock", "blocking", "waiting on me", "before [Person/team] can"
- Task has `[[@Person]]` + verb like "review," "approve," "respond," "unblock"

**BossBoost** — **+400** if the source breadcrumb references a 1:1 with a Tier 1-2 person from `vault-config.people.tiers`:
- Source includes `[[<vault.layout.oneOnOnes>/[Tier 1-2 Name]/...]]`
- Meeting notes where Tier 1-2 attended (heuristic: filename contains "ELT", "SLT", "Leadership" → +200 partial boost)

If the boss 1:1 breadcrumb is older than 60 days, halve the BossBoost.

Round all scores to integers. Rank descending. Ties: earlier due date first.

### 3. Generate the HTML

Read `template.html` (alongside this `SKILL.md`). Replace two placeholders:

1. `const TASKS = [];` → full JSON array of task objects (schema below)
2. `const TRIAGE_META = {};` → metadata object

**Task object schema:**

```json
{
  "id": "a1b2c3d4",
  "text": "Task description",
  "raw": "- [ ] Task description ⏫ #tag 📅 2026-04-17 → [[Source]]",
  "workType": "Strategic",
  "priority": "high",
  "tags": ["#tag"],
  "dueDate": "2026-04-17",
  "daysUntilDue": 0,
  "people": ["@Person"],
  "sources": ["Calendar/Meetings/Source/2026-04-09"],
  "flags": {
    "bossAttributed": true,
    "isBlocker": false,
    "isOverdue": false,
    "isStale": false,
    "waiting": false,
    "someday": false
  },
  "scoring": {
    "base": 800,
    "deadlineBoost": 400,
    "stalePenalty": 0,
    "blockerBoost": 0,
    "bossBoost": 200,
    "dailyScore": 1400
  },
  "reasoning": "Due today · Boss-attributed · Strategic"
}
```

**Triage meta:**

```json
{
  "today": "2026-04-17",
  "totalOpen": 72,
  "totalWaiting": 8,
  "totalSomeday": 15,
  "focusCap": 5,
  "directReports": ["Riley Cohen", "Nia Okonkwo", "Tom Bauer"]
}
```

The `directReports` array populates the Delegate modal dropdown. Pull from `vault-config.people.directReports` (use the `name` field of each entry).

Save the populated HTML to vault root as `📋 Action Items Triage.html`. Tell the user to open it and give the one-line return instruction ("paste the decisions JSON and say: apply decisions").

### 4. Brief summary before pointing to the HTML

- Total tasks surfaced for triage
- How many are overdue
- How many are boss-attributed (Tier 1-2)
- How many are blockers (potentially unblocking others)
- Top 3 scored tasks as a sanity check

Keep it tight — this is a daily 5–10 min ritual, not a debrief.

---

## Phase 2 — Apply decisions back to the vault

Triggered when the user says "apply decisions", "apply the triage", or similar after finishing the HTML. They'll have `action-items-decisions.json` in Downloads, OR they'll paste the JSON directly.

### Decisions JSON schema

```json
{
  "triageDate": "2026-04-17",
  "decisions": [
    { "id": "a1b2c3d4", "action": "keep" },
    { "id": "e5f6g7h8", "action": "defer", "newDueDate": "2026-04-24" },
    { "id": "i9j0k1l2", "action": "delegate", "to": "@Person Name" },
    { "id": "m3n4o5p6", "action": "kill", "reason": "superseded by Project X" },
    { "id": "q7r8s9t0", "action": "complete" }
  ],
  "focus": ["a1b2c3d4", "x7y8z9w0", "p0o9i8u7"]
}
```

### How to apply each action

Read `📋 Action Items.md` fresh. For each decision:

**`keep`** — no change.

**`defer`** — find the task's `- [ ]` line (match by id / text). Replace `📅 YYYY-MM-DD` with the new date. Leave everything else.

**`delegate`** — append ` @delegate [[@Name]]` to the task line (before the `→` breadcrumb if present). Add a sub-item:
```
  - Delegated YYYY-MM-DD. Follow up if no movement by YYYY-MM-DD.
```
Follow-up date = triage date + 5 business days.

**`kill`** — remove from `## Open`. Append to `## Completed`:
```
- [x] Task description 📅 original-due-date ✅ triage-date #cancelled #original-tag → [[Original Source]]
  - Cancelled: {reason}
```

**`complete`** — remove from `## Open`. Append to `## Completed`:
```
- [x] Task description 📅 original-due-date ✅ triage-date #original-tag → [[Original Source]]
```
No `#cancelled` tag, no sub-item note. For tasks already done at triage time. If the decision has a `note` field, it becomes a sub-item: `- Completed: {note}`.

**`focus`** — repopulate `## 🎯 This Week` at the top with exactly these tasks (in order), copying their full `- [ ]` line from Open. Leave the original in Open — `🎯 This Week` is a pointer, not a move.

### Preservation rules

- **Never touch** frontmatter, section headers, `## Waiting On Others`, or `## Someday / Maybe` (unless the user explicitly triaged something from those — then follow the same rules).
- **Preserve breadcrumbs** (`→ [[...]]`) on every edited line.
- **Always edit** `📋 Action Items.md` in place. Don't create a new file.

### Verification gate

Re-read `📋 Action Items.md` and verify:
1. `🎯 This Week` has exactly the focus IDs requested (3–5)
2. All `defer` show the new due date
3. All `delegate` show `@delegate [[@Name]]`
4. All `kill` are absent from `## Open` and present in `## Completed` with `#cancelled`
5. All `complete` are absent from `## Open` and present in `## Completed` without `#cancelled`

Report a one-line summary per action: "Kept 22 · Deferred 8 · Delegated 4 · Completed 3 · Killed 7 · Focused 5."

---

## UX principles for the HTML

- **Keyboard-first.** Keys: `K` Keep / `D` Defer / `G` Delegate / `X` Kill / `C` Complete. Arrow keys navigate. Enter confirms. `1` / `2` / `3` switch tabs.
- **Progress always visible.** Top bar: "N of M triaged · X min elapsed."
- **Undo.** Last 5 decisions can be undone with `U` or backspace.
- **No nags, no confirmations** — has to feel frictionless to sustain daily.
- **Reasoning visible but collapsed.** Each card shows a one-line "why it scored this high" under the task text. Click to expand full breakdown.
- **Bailout.** "I'm done" button at any point exports decisions for triaged-so-far tasks; untriaged default to `keep`.

## Edge cases

- **Empty backlog.** Say so plainly — no HTML generated.
- **No open tasks in one work type.** Hide that column in Balance; don't break layout.
- **Duplicate task text after `/action-items-compress`.** Shouldn't happen, but if it does: use the first occurrence and skip the rest; warn the user.
- **Task has no due date.** DeadlineBoost = 0; rely on BASE and other boosts.
- **Focus cap conflict.** If the user picks 6 focus tasks in HTML, the page blocks the 6th with "cap = 5." In Phase 2, trust the JSON — if 6 come through, apply the first 5 and warn.
- **Decision JSON references a task id not in current Action Items.md.** Skip with a warning. Don't fail the batch.

## File locations

- Backlog: `📋 Action Items.md` at vault root
- Template: alongside this `SKILL.md` as `template.html`
- Output HTML: `📋 Action Items Triage.html` at vault root
- Decisions JSON: typically `~/Downloads/action-items-decisions.json` (default browser download)

## Customizing visuals

The bundled `template.html` uses neutral colors (`#2563eb` blue accent) and system fonts. To customize:

1. Edit the `:root` CSS variables (`--color-accent`, `--color-accent-dark`)
2. Add a `<link>` to Google Fonts and update the `font-family` declarations
3. The `--bg-light`, `--border`, `--muted-*`, `--danger`, and `--warn` variables control the rest of the palette
