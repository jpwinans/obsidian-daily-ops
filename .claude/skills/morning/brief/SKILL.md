---
name: morning-brief
description: |
  Lighter morning briefing. Scans vault state and overnight Slack only —
  no Gmail triage, no risk-digest deep dive. Use when you want a fast
  start-of-day summary without the full /morning:start synthesis.
  Triggers: "quick morning brief", "lighter daily briefing", "/morning brief".
model: claude-sonnet-4-6
allowed-tools: >
  Read Write Edit Glob Grep
  mcp__claude_ai_Slack__slack_read_channel
  mcp__claude_ai_Slack__slack_search_public_and_private
  mcp__claude_ai_Notion__notion-search
---

# /morning:brief — Lighter daily briefing

A faster, narrower version of `/morning:start`. Use when you want overnight Slack signal + vault state (tasks, blockers, stale relationships) without the full risk digest or email triage.

## Input

No arguments. Uses today's date.

## Instructions

### Step 1 — Read context

1. Load `vault-config` for `channels`, `personTiers`, `projects`, `conventions`, `vault.layout`.
2. Load `obsidian-markdown` for syntax reference.
3. Read the Daily Note template at `<vault.layout.templates>/Daily Note.md`.

### Step 2 — Check the daily note

Compute today's path. If missing, create from template. If present, preserve content.

### Step 3 — Scan Slack via slack-triage

Lookback: 24 hours (or 72 hours on Monday).

Pass channel groups from `vault-config.channels` and `personTiers` to `slack-triage`. It returns classified items.

For deploy/alerts channels, summarize successful deploys in a one-liner; flag failures/rollbacks prominently with a permalink.

### Step 4 — Optional Notion check

If Notion MCP is connected, call `notion-search` with `content_search_mode: "workspace_search"` for each project name in `vault-config.projects` to surface recently updated pages. If Notion is unavailable, skip.

### Step 5 — Scan vault state

1. Overdue tasks (incomplete `- [ ]` with `📅` before today).
2. Tasks due today.
3. Stale relationships (person notes with `last-contact` and `contact-frequency` overdue).
4. Inbox count (`<vault.layout.inbox>`).
5. Today's meetings already created.

### Step 6 — Generate Morning Briefing section

Structure:

```markdown
## Morning Briefing

> [!tip] Top 3 Items Needing Attention
> 1. **[Most important]** — [why and what to do]
> 2. **[Second]** — [context]
> 3. **[Third]** — [context]

### Overnight Alerts
[Slack summary — flag urgent. "No critical overnight activity." if nothing.]

### Deploy Status
[One-liner from deploy/alerts group, or "No deploys in the last 24h."]

### Tasks

**Overdue ([count]):**
- [ ] [Task] 📅 [date] — from [[source]]

**Due Today ([count]):**
- [ ] [Task] 📅 [today] — from [[source]]

### Stale Relationships

| Person | Last Contact | Days Overdue |
|--------|-------------|--------------|
| [[@Name]] | YYYY-MM-DD | N |

### Inbox
[N items] — [list briefly or "Inbox clear"]
```

### Step 7 — Write the daily note

Same insertion rules as `/morning:start`: insert Morning Briefing after frontmatter, preserve existing content.

### Step 8 — Stdout summary

- Overdue / due-today / blocker counts
- Critical overnight alerts (or note)
- Stale relationships flagged
- Inbox count
- Channels scanned
- MCP gaps

## Notes

- Same "ownership classification" rule as `/morning:start` — never present someone else's task as the user's.
- Use wikilinks for vault references. People as `[[@Name]]`.
- The briefing should be scannable in under 90 seconds.
