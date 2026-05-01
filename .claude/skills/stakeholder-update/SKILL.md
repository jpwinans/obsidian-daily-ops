---
name: stakeholder-update
description: |
  Generates audience-specific status updates from the latest weekly review,
  active project notes, and recent decisions. Audiences match what's
  configured in CLAUDE.md (`manager`, `team`, `leadership`, etc.). Output
  to stdout. Triggers: "draft a stakeholder update", "/stakeholder update".
model: claude-sonnet-4-6
argument-hint: "[manager | team | leadership | all] (defaults to all)"
allowed-tools: Read Glob Grep
---

# /stakeholder-update — Drafted updates for stakeholder audiences

Generate audience-appropriate status updates from the week's activity and project status.

## Input

`$ARGUMENTS` = audience name. Common values: `manager`, `team`, `leadership`, `all`. Defaults to `all`.

The list of recognized audiences comes from `vault-config.conventions.stakeholderAudiences`. The default set if not configured: `manager`, `team`, `leadership`.

## Instructions

### Step 1 — Read context

1. Load `vault-config` for `vault.layout`, `personTiers`, `directReports`, `stakeholders`, `projects`, `conventions`. Apply the documented `vault-config` contract.
2. Load `obsidian-markdown` for syntax.

### Step 2 — Gather source material

1. **Most recent weekly review:** read the latest file in `<vault.layout.weekly>/` (sorted by date desc).
2. **Active project notes:** read project notes from `<vault.layout.projects>/` where `status` is not `complete` or `archived`.
3. **OKR status:** read `<vault.layout.outcomes>/Current OKRs.md` if it exists.
4. **Recent decisions:** search for notes tagged `#decision` from the past week.

### Step 3 — Generate audience-specific drafts

Match the requested audience(s) and emit drafts. Skip audiences the user didn't request. For `all`, emit all configured audiences separated by `---`.

#### Manager / 1:1 update

Detailed status for the user's manager (Tier 2 from `vault-config.people.tiers`).

```markdown
## Status Update for [Manager Name] — Week of YYYY-MM-DD

### Progress
- **[Initiative]:** [Detailed status — what shipped, what moved, what's next]
- **[Initiative]:** [Detailed status]

### Blockers & Risks
- [Blocker description and what's needed to resolve]

### Org & People
- [Team dynamics, hiring, 1:1 insights — anything the manager should know]

### Strategic Input Needed
- [Questions or topics where the manager's guidance would help]

### Discussion Items
- [Topics for the 1:1 conversation]
```

Rules: candid, include org/people topics, frame asks clearly. The manager values ownership — show driving, not just reporting.

#### Team update

Sprint-focused update for the user's team. Use the team name from `vault-config.directReports`'s most-mentioned team or whatever team appears in the user's person note.

```markdown
## Team Update — Week of YYYY-MM-DD

### What Shipped
- [Completed items and wins]

### What's Next
- [Upcoming priorities for the coming week]

### Who's Blocked
- [Team members with blockers and what's needed]

### Wins & Celebrations
- [Shout-outs, milestones, good work to recognize]
```

Rules: action-oriented, name people for both blockers and celebrations, short and scannable.

#### Leadership / executive update

Executive summary for senior leadership (Tier 1-3 audience).

```markdown
## Leadership Update — Week of YYYY-MM-DD

**[Org area] — [[@User Name]]**

- **[Initiative]:** [1-sentence business impact summary with metric if available]
- **[Initiative]:** [1-sentence status with risk flag if yellow/red]
- **[Initiative]:** [Progress or milestone]
- **Risk/Blocker:** [If any — otherwise omit]
- **Decision needed:** [If any — otherwise omit]
```

Rules: 3–5 bullets max. Lead with business impact, not technical details. Flag risks explicitly. Include metrics where available.

### Step 4 — Output

All drafts go to stdout for review. Do NOT write to files or send to Slack automatically.

After outputting, note: "These drafts are ready for your review. When you're satisfied, post them manually or send via Slack MCP if you'd like."

## Notes

- Output to stdout only — does not auto-send.
- Wikilinks for internal vault references. People as `[[@Person]]`.
- If no weekly review exists yet, note this and suggest running `/rollup-weekly` first.
- Adapt tone to audience: executive for leadership, candid for manager, motivating for team.
- Omit sections with no content rather than leaving them empty.
- If a custom audience name (not `manager`/`team`/`leadership`) is passed, treat it as a manager-style update aimed at the named role and ask for clarification if the name doesn't map to anyone in `personTiers`.
