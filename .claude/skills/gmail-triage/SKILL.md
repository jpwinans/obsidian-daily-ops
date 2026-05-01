---
name: gmail-triage
description: |
  GTD inbox triage. Reads Gmail, classifies each thread by ownership and
  sender importance, and presents a recommendation table mapping threads
  to the user's GTD label structure (by NAME, not ID). Read-only —
  applies labels manually. Triggers: "triage my inbox", "gmail triage",
  "/gmail triage".
model: claude-sonnet-4-6
allowed-tools: >
  Read
  mcp__claude_ai_Gmail__gmail_search_messages
  mcp__claude_ai_Gmail__gmail_list_labels
  mcp__claude_ai_Gmail__gmail_read_message
---

# /gmail-triage — GTD inbox triage

Read the Gmail inbox, classify each thread by ownership and sender importance, recommend GTD labels.

## Input

No arguments. Scans the inbox by default.

## Why this skill exists

Email is the noisiest input. Most inbox messages are FYI, automated, or someone else's action item. Use this to surface the 3–5 emails that actually need a direct response or decision — and ignore the rest. Same ownership-classification logic as `/morning-start` and `/blocker-scan`, applied to email.

## Instructions

### Step 1 — Read context

1. Load `vault-config` for `personTiers`, `directReports`, `stakeholders`, `labels`. Apply the documented `vault-config` contract — if no `## Gmail GTD Labels` section, the skill cannot recommend labels; surface the gap and exit.
2. Note today's date for time-sensitivity.

### Step 2 — Delegate to gmail-gtd-triage

Invoke the `gmail-gtd-triage` helper skill with:
- `labels`: from `vault-config.labels`
- `personTiers`: from `vault-config.people.personTiers`
- `query`: `in:inbox newer_than:3d`
- `maxThreads`: 30

Note: do NOT use `is:unread` as the primary filter — many users have filters that route auto-labeled messages (GitHub, Jira, etc.) past the inbox. Always anchor on `in:inbox`.

It returns classified items with recommended labels (matched by name from CLAUDE.md, not by ID).

### Step 3 — Compute urgency score for sort order

For Next Action items, score:

```
Urgency = PersonTierWeight + TimeSensitivity + DirectnessOfAsk
```

- **PersonTierWeight**: from `vault-config.people.tiers` (Tier 1 = 5, Tier 2 = 4, ...)
- **TimeSensitivity**: today = 3, this week = 2, no deadline = 0
- **DirectnessOfAsk**: explicit ask to user = 3, implicit = 1, FYI = 0

Sort Next Actions by urgency descending.

### Step 4 — Present triage summary to stdout

```markdown
# Gmail Triage — YYYY-MM-DD HH:MM

**Scanned:** N inbox messages (last 3 days) | **Actionable:** N | **Archive:** N

---

## GTD Classification

| # | From | Subject | Starred? | Recommended Label | Rationale |
|---|------|---------|----------|-------------------|-----------|
| 1 | [Sender] | [Subject] | ★ | **[label name from CLAUDE.md]** | [why] |

---

## Summary by Label

| GTD Label | Count | Items |
|-----------|-------|-------|
| **[label]** | N | [subjects] |

*Omit rows with zero items.*

---

## Details (Next Actions only)

**1. [Subject]** — From: [sender] — [date]
> [1–2 sentence summary of what's being asked and what the user needs to do]
```

Recommended-label values come from the user's `## Gmail GTD Labels` section, by name. If `gmail-gtd-triage` returned `recommendedLabel: null` (no matching label name), say so explicitly with the closest fit suggestion.

### Step 5 — Offer next steps

Ask:

> Want me to open any of these threads for a deeper read? Or draft a reply to any of the Next Actions?

This lets the user act immediately on the highest-priority items.

## Notes

- **Read-only.** The Gmail MCP usually cannot apply labels — recommend, the user applies manually.
- **Ownership classification is key.** Not every email TO the user is FOR the user. Apply the same "who is being asked?" logic as `/morning-start`.
- **Don't over-classify as Next Action.** GTD works because the Next Actions list is short. If everything is a Next Action, nothing is. Err toward Reference when in doubt.
- **Automated messages are never Next Actions.** Jira notifications, GitHub PRs, deploy alerts, calendar invites without prep, newsletters → archive or reference.
- **Respect the tiers.** A vague FYI from a Tier 1-2 person is still worth flagging; a direct ask from an unknown sender isn't necessarily urgent.
- **Privacy:** don't output full email bodies — 1–2 sentence summaries only.
