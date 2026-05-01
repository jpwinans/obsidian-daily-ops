---
name: blocker-scan
description: |
  Risk digest. Scans Slack channels and vault state for blockers, risks,
  and items needing attention. Classifies by ownership (action / awareness)
  and ages by tier (stale / aging / recent / fresh). Output to stdout —
  does not write to any file. Triggers: "scan for blockers", "risk digest",
  "/blocker scan".
model: claude-sonnet-4-6
allowed-tools: >
  Read Grep Glob
  mcp__claude_ai_Slack__slack_read_channel
  mcp__claude_ai_Slack__slack_read_thread
  mcp__claude_ai_Slack__slack_search_public_and_private
---

# /blocker-scan — Risk detection

Scan Slack channels and vault state for blockers, risks, and items needing attention. Output a structured risk digest to stdout. Does not write to any file.

## Input

No arguments.

## Instructions

### Step 1 — Read context

1. Load `vault-config` for `channels`, `personTiers`, `directReports`, `projects`, `vault.layout`. Apply the documented `vault-config` contract — if no `## Channels` section, skip the Slack scan and run vault-state-only.

### Step 2 — Scan Slack via slack-triage

Lookback:
- Monday → 72 hours
- Other weekdays → 24 hours

Pass `vault-config.channels` (all groups), `personTiers`, and the lookback to `slack-triage`. It returns classified items (action / awareness / skip), age-tiered, severity-scored, plus a `deploySummary`.

### Step 3 — Scan vault state

1. **Open blockers:** grep for tasks tagged `@blocked`.
2. **Overdue tasks:** tasks with due dates that have passed.
3. **Recent meeting blockers:** check recent files in `<vault.layout.meetings>/` for mentioned blockers or risks.

### Step 4 — Cross-reference

For each flagged Slack item, check connections to:
- Active projects in `vault-config.projects` or `<vault.layout.projects>/`
- People in `directReports`/`stakeholders`
- Teams in `<vault.layout.teams>/`

Add a `connection: [[...]]` line where applicable.

### Step 5 — Generate risk digest to stdout

```markdown
# Risk Digest — YYYY-MM-DD HH:MM

## 🔴 Stale Blockers — Over 72 Hours Old

> These have been sitting longest. If they haven't moved, something is wrong.

### [Issue Title]
- **Age:** [N days / hours since first surfaced]
- **Severity:** Critical / Warning
- **Source:** [Channel] — [first seen timestamp]
- **Message:** [Summary or quote]
- **Connection:** Links to [[Project]] or [[@Person]]
- **Suggested action:** [What the user should do]

---

## 🟠 Aging Blockers — Over 48 Hours Old

> Getting stale. Needs attention before they become critical.

[items...]

---

## 🟡 Recent Blockers — Yesterday

> Surfaced in the last 24–48 hours. Monitor closely.

[items...]

---

## ℹ️ Info (Awareness Only)

- [Brief item] — [source] — [age]

---

## Vault State

- **Overdue tasks:** [count] ([list top 3 if any])
- **Blocked items:** [count] ([list if any])
- **Recent decisions pending follow-up:** [any from recent meetings]

---

*Scan completed at YYYY-MM-DD HH:MM. Channels scanned: [list all]*
```

### Severity assessment

Age tiers are the primary sort key. Within each tier, sort by severity:

| Age Tier | Threshold | Section label |
|----------|-----------|---------------|
| 🔴 Stale | > 72 hours since first surfaced | Over 72 Hours Old |
| 🟠 Aging | > 48 hours since first surfaced | Over 48 Hours Old |
| 🟡 Recent | 24–48 hours | Yesterday |
| ℹ️ Fresh | Under 24 hours | Info only |

Severity:
- **Critical:** production incidents, data issues, security concerns, urgent help requests
- **Warning:** build failures, repeated errors, blocked 2+ days, performance degradation
- **Info:** general complaints, minor issues, FYI mentions

For vault tasks, the `📅` due date is when it was expected — if overdue 3+ days, treat as 🔴 Stale.

If a section has no items, omit it entirely.

## Notes

- Output to **stdout only** — does not write to any file.
- If the Slack MCP is unavailable, skip Slack sections and note the gap. Always include the Vault State section even if Slack scanning fails.
- Be concise — quick scan, not deep investigation.
- "Action required" items include a **Suggested action**. Awareness items go in the Info section — never frame them as the user's tasks.
