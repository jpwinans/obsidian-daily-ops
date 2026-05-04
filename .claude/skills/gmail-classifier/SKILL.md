---
name: gmail-classifier
description: |
  Encapsulates Gmail inbox classification — reads inbox, classifies each
  thread by ownership and sender importance, and recommends a label
  assignment by NAME (not ID) from the user's configured label set.
  Auto-loaded by /morning-start for the email-triage portion of the briefing.
user-invocable: false
model: inherit
allowed-tools: >
  Read
  mcp__claude_ai_Gmail__gmail_search_messages
  mcp__claude_ai_Gmail__gmail_list_labels
  mcp__claude_ai_Gmail__gmail_read_message
---

# gmail-classifier — Gmail inbox classification

You classify Gmail threads into the user's configured label structure. Workflow skills load you when they need email triage.

## When to use this skill

Other skills invoke you with a parsed config from `vault-config` and a search query. Do not run on your own.

## Input you receive

- `labels` — list of `{ label, purpose }` from `vault-config.labels` (the user's label hierarchy by name).
- `personTiers` — sender-importance map from `vault-config.people.personTiers`.
- `query` — Gmail search query (default: `in:inbox newer_than:3d`).
- `maxThreads` — cap (default 30).

## What you do

### 1. Confirm labels exist
Call `gmail_list_labels` once to confirm each label name from `labels` exists in the user's Gmail. If any are missing, surface a warning ("Label `Waiting` not found in Gmail — will recommend but cannot apply").

### 2. Search inbox
Call `gmail_search_messages` with the provided query (cap `maxThreads`). Do not use `is:unread` alone — most users have filters that route automated mail (GitHub, Jira, etc.) past the inbox. Scan only what actually lands in `inbox`.

### 3. Read selectively
For threads where the snippet is ambiguous OR the sender matches Tier 1-4 in `personTiers`, call `gmail_read_message` for the full body. Skip obvious automated messages (HR reminders, security training, newsletters) — classify from subject/snippet alone.

### 4. Classify ownership

For each thread, decide:

- **action-reply** — A direct ask from a Tier 1-2 person, or a question/blocker the user must respond to in writing.
- **action-task** — Form, training, doc review, or other computer-task with a deadline.
- **action-discuss** — Item to raise in an upcoming meeting or 1:1.
- **action-call** — Requires a phone or video call.
- **waiting** — Status update on something the user delegated; ball in someone else's court.
- **delegated** — Handed off to someone, tracking completion.
- **reference** — Useful info, no action (FYI, newsletters worth keeping).
- **someday** — Not urgent, review later.
- **archive** — Duplicate reminders, low-value automation, old confirmations.

### 5. Map ownership to a specific label

Pick the closest match from the user's `labels`. Use label names from CLAUDE.md, not hardcoded ones. Common mappings (adapt to whatever names the user wrote):

| Ownership | Match label name containing |
|-----------|------------------------------|
| action-reply | `Reply` / `Email` / `Action/Reply` |
| action-task | `Review` / `Computer` / `Action/Task` |
| action-discuss | `Agenda` / `Discuss` / `Action/Discuss` |
| action-call | `Call` / `Calls` |
| waiting | `Waiting` |
| delegated | `Delegated` |
| reference | `Reference` |
| someday | `Someday` |

If no label name in CLAUDE.md matches, return `recommendedLabel: null` and a note explaining what the closest fit was.

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
      "ownership": "action-reply",
      "recommendedLabel": "Action/Reply",
      "actionSummary": "1-line: what the user needs to do",
      "deadline": "YYYY-MM-DD" | null,
      "linksToVault": "[[@Person]] or [[Project]]" | null
    },
    ...
  ],
  "stats": {
    "actionable": 4,
    "waiting": 2,
    "reference": 5,
    "archiveCandidates": 12
  }
}
```

## Important notes

- The Gmail MCP integration is read-only in most setups — labels must usually be applied manually. Output recommendations, not commands.
- Deadlines mentioned in snippets ("by Friday") should be converted to absolute dates in the output.
- If the inbox is empty or all archive-worthy, return `items: []` and `stats.actionable: 0`. The calling skill will say "Inbox clear."
