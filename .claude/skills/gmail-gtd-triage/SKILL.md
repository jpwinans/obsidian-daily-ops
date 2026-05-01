---
name: gmail-gtd-triage
description: |
  Encapsulates Gmail GTD classification — reads inbox, classifies each
  thread into the user's GTD label structure based on ownership and
  sender importance, and recommends label assignments by NAME (not ID).
  Auto-loaded by /gmail-triage and the email portion of /morning-start.
user-invocable: false
model: inherit
allowed-tools: >
  Read
  mcp__claude_ai_Gmail__gmail_search_messages
  mcp__claude_ai_Gmail__gmail_list_labels
  mcp__claude_ai_Gmail__gmail_read_message
---

# gmail-gtd-triage — GTD inbox classification

You classify Gmail threads into the user's GTD label structure. Workflow skills load you when they need email triage.

## When to use this skill

Other skills invoke you with a parsed config from `vault-config` and a search query. Do not run on your own.

## Input you receive

- `labels` — list of `{ label, purpose }` from `vault-config.labels` (the user's GTD label hierarchy by name).
- `personTiers` — sender-importance map from `vault-config.people.personTiers`.
- `query` — Gmail search query (default: `in:inbox newer_than:3d`).
- `maxThreads` — cap (default 30).

## What you do

### 1. Confirm labels exist
Call `gmail_list_labels` once to confirm each label name from `labels` exists in the user's Gmail. If any are missing, surface a warning ("Label `📥 GTD/2 - Waiting For` not found in Gmail — will recommend but cannot apply").

### 2. Search inbox
Call `gmail_search_messages` with the provided query (cap `maxThreads`). Do not use `is:unread` alone — most users have filters that route automated mail (GitHub, Jira, etc.) past the inbox. Scan only what actually lands in `inbox`.

### 3. Read selectively
For threads where the snippet is ambiguous OR the sender matches Tier 1-4 in `personTiers`, call `gmail_read_message` for the full body. Skip obvious automated messages (HR reminders, security training, newsletters) — classify from subject/snippet alone.

### 4. Classify ownership

For each thread, decide:

- **next-action** — A direct ask from a Tier 1-2 person, a blocker/question from a direct report, automated forms/training with a deadline, or a review request the user owns.
- **waiting-for** — Status update on something the user delegated; ball in someone else's court.
- **delegated** — Handed off to someone, tracking completion.
- **reference** — Useful info, no action (FYI, newsletters worth keeping).
- **someday-maybe** — Not urgent, review later.
- **archive** — Duplicate reminders, low-value automation, old confirmations.

### 5. Map ownership to a specific label

Pick the closest match from the user's `labels`. Use label names from CLAUDE.md, not hardcoded ones. Common mappings (adapt to whatever names the user wrote):

| Ownership | Match label name containing |
|-----------|------------------------------|
| next-action (reply needed) | `Next Actions/@Email` |
| next-action (computer task) | `Next Actions/@Computer` |
| next-action (raise in meeting) | `Next Actions/@Agenda` |
| next-action (call someone) | `Next Actions/@Calls` |
| waiting-for | `Waiting For` |
| delegated | `Delegated` |
| reference | `Reference` |
| someday-maybe | `Someday Maybe` |

If no label name in CLAUDE.md matches, return `recommendedLabel: null` and a note explaining what the closest fit was.

### 6. Tag each next-action with a context

For ownership `next-action`, infer the right context tag (`@Email`, `@Computer`, `@Agenda`, `@Calls`) from what the action requires. A meeting follow-up question goes to `@Agenda`. A form to fill out goes to `@Computer`. A reply to a person goes to `@Email`.

## Output you return

```json
{
  "labelsConfirmed": ["...", ...],
  "labelsMissing": ["..."],
  "totalScanned": 23,
  "items": [
    {
      "threadId": "1",
      "from": "Sender Name <addr>",
      "senderTier": 2,
      "subject": "...",
      "snippet": "...",
      "ownership": "next-action",
      "recommendedLabel": "📥 GTD/1 - Next Actions/@Email",
      "actionSummary": "1-line: what the user needs to do",
      "deadline": "YYYY-MM-DD" | null,
      "linksToVault": "[[@Person]] or [[Project]]" | null
    },
    ...
  ],
  "stats": {
    "actionable": 4,
    "waitingFor": 2,
    "reference": 5,
    "archiveCandidates": 12
  }
}
```

## Important notes

- The Gmail MCP integration is read-only in most setups — labels must usually be applied manually. Output recommendations, not commands.
- Deadlines mentioned in snippets ("by Friday") should be converted to absolute dates in the output.
- If the inbox is empty or all archive-worthy, return `items: []` and `stats.actionable: 0`. The calling skill will say "Inbox clear."
