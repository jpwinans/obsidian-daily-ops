---
name: meeting-ingest
description: |
  Finds meeting transcript emails in Gmail (Gemini Notes, Google Meet
  transcripts) within a lookback window, extracts the transcript content,
  and uses it to populate or enrich the corresponding vault meeting note
  with synthesized summaries, decisions, and action items. Never pastes
  raw transcripts. Triggers: "ingest meeting transcripts", "process new
  meeting recordings", "/meeting ingest [days]".
model: claude-sonnet-4-6
argument-hint: "[lookback-days, default 7]"
allowed-tools: >
  Read Write Edit Glob Grep
  mcp__claude_ai_Gmail__gmail_search_messages
  mcp__claude_ai_Gmail__gmail_search_threads
  mcp__claude_ai_Gmail__gmail_read_message
  mcp__claude_ai_Gmail__gmail_get_thread
  mcp__claude_ai_Google_Drive__read_file_content
  mcp__claude_ai_Google_Drive__get_file_metadata
  mcp__claude_ai_Google_Drive__download_file_content
---

# /meeting-ingest — Process meeting transcripts into vault notes

Find transcript emails in Gmail, extract content, and populate or enrich the corresponding vault meeting note. Never paste raw transcripts — use them only as source material.

## Input

`$ARGUMENTS` = optional lookback in days (e.g., `3`, `7`). Defaults to `7`.

## Instructions

### Step 1 — Read context

1. Load `vault-config` for `vault.layout`, `personTiers`, `projects`. Apply the documented `vault-config` contract.
2. Load `obsidian-markdown` for syntax.
3. Load `meeting-notes` for the structure standard.

### Step 2 — Search Gmail for transcript emails

Use `gmail_search_messages` (or `search_threads`) with these queries (lookback `N` days):

**Primary senders:**
- `from:gemini-notes@google.com` — Gemini AI meeting notes
- `from:meet-recordings-noreply@google.com` — Google Meet recordings
- `from:meetings-noreply@google.com` — Meet 1:1 transcripts

Run:
1. `from:gemini-notes@google.com newer_than:{N}d`
2. `from:meet-recordings-noreply@google.com newer_than:{N}d`
3. `from:meetings-noreply@google.com newer_than:{N}d`

Combine + dedupe by thread ID. For each: capture thread ID, message ID, subject, date, snippet, attachment/link metadata.

**If zero matching emails are found in the lookback window:** stop here and report "No new transcript emails in the last N days. Nothing to ingest." Do not proceed to Step 3. This is expected on fresh clones, low-activity weeks, or when the user hasn't yet enabled Gemini Notes / Meet recording.

### Step 3 — Extract transcript content

For each email, use `gmail_read_message` (or `get_thread`) for the full body. Strategies:

1. **Inline content:** some Gemini emails embed the notes/transcript directly. Extract substantive text — discussion points, action items, decisions — verbatim for processing.
2. **Linked Google Doc:** if body contains `docs.google.com/document/d/[DOC_ID]`, extract the doc ID and use Google Drive `read_file_content` to fetch full text.
3. **Linked Drive file:** if body links to `drive.google.com/file/d/[FILE_ID]`, extract the file ID. Use `get_file_metadata` to confirm readable, then `read_file_content` or `download_file_content`.

If content cannot be retrieved (access denied, link expired), note the failure and skip — do not create a stub note.

### Step 4 — Parse meeting metadata

From the email subject, body, and transcript:
- **Meeting name** — usually in the subject ("Your meeting notes from 'X'", "Transcript: X"). Strip the prefix to get the clean name.
- **Meeting date** — from the email date or explicit date in the subject/body. Convert to `YYYY-MM-DD`.
- **Attendees** — listed in body or transcript header. Map names to `<vault.layout.people>/@First Last.md`.
- **Recurring vs. ad hoc** — check `<vault.layout.meetings>/[Name]/` for prior instances.

### Step 5 — Classify meeting type

Same logic as `/prep-day`:
- **1-1:** title contains `1:1`, `1-1`, `1 on 1`, OR exactly 2 attendees (user + one person)
- **Recurring:** `<vault.layout.meetings>/[Name]/` exists with prior instances AND not 1-1
- **Ad hoc:** no prior vault folder AND not 1-1

### Step 6 — Find or create the vault note

**Target paths:**
- 1-1: `<vault.layout.oneOnOnes>/[Person Name]/YYYY-MM-DD.md`
- Meeting / Ad hoc: `<vault.layout.meetings>/[Meeting Name]/YYYY-MM-DD.md`

If the note exists, read it — proceed to Step 7 to enrich.

If it doesn't exist, invoke the appropriate prep skill first:
- 1-1 → `prep-1on1` with the person's name
- Recurring → `prep-meeting` with the meeting name
- Ad hoc → `prep-meeting` with `[Meeting Name] | as Ad Hoc`

After prep creates the note, read it before Step 7.

### Step 7 — Process the transcript

Using the transcript as source material, extract:

#### Action Items
- **User's tasks:** items assigned to the user. Use Tasks-plugin format:
  ```
  - [ ] Task description ⏫ 📅 YYYY-MM-DD
  ```
  ⏫ for urgent/time-sensitive, 🔼 for standard. Add `@context`.
- **Others' tasks:** items assigned to other people. List without checkboxes:
  ```
  - [[@Person]] — what they own
  ```

#### Decisions
Statements of record — choices made, paths agreed, things confirmed:

| Decision | Owner |
|----------|-------|
| What was decided #decision | [[@Owner]] |

#### Discussion Topics
Synthesize each major topic into a structured section:
- Concise heading (what was discussed, not a vague label)
- 2–5 bullets of key points framed for future reference
- Important context, risks, or open questions

Do NOT quote the transcript. Write synthesized notes that make sense reading them in 3 months.

#### Observations
1–3 observations from the user's perspective — what this means for their projects, direct reports, or broader priorities. Draw on `vault-config.projects` and recent vault activity.

### Step 8 — Update the vault note

Merge extracted content into the existing structure:

- **Empty section (stub from prep):** fill in.
- **Section has prep content:** augment — keep prep-populated context, add transcript-derived substance below. Don't delete prep content.
- **Frontmatter:** update `status` `draft` → `active`. Ensure all attendees in `attendees:`. Add new `related:` links.
- **Type field:** `Recurring` or `Ad-hoc` per Step 5.
- **Do NOT add a Raw Transcript section.** Source material only; doesn't go in the note.

### Step 9 — Output summary

```
## Meeting Transcript Ingest — YYYY-MM-DD

**Emails found:** N (Gemini: N, Meet: N)
**Processed:** N
**Skipped:** N (reason: access denied / no matching content / already up to date)

| Meeting | Date | Type | Note Path | Status |
|---------|------|------|-----------|--------|
| ... | YYYY-MM-DD | Recurring | <vault.layout.meetings>/.../YYYY-MM-DD.md | ✓ updated |
```

## Notes

- **No emails found is normal**, not a failure. Report cleanly and exit.
- **Gmail label structure is not assumed** — this skill reads message *bodies*, not labels. It works on any Gmail account that receives Gemini Notes or Google Meet transcript emails.
- **Never paste the raw transcript** — not inline, not in callouts, not collapsed. Synthesize only.
- **Ownership classification:** apply tier mapping. Not every action discussed is the user's. Only items where the user is the named owner or clearly responsible get checkboxes. Everyone else's go under "Action Items (Others)".
- Wikilink all people and projects.
- **If Drive content is inaccessible:** skip the email rather than creating an empty or hallucinated note. Report the skip.
- **Deduplication:** if two emails reference the same meeting on the same date (Meet transcript + Gemini summary), merge them — both as source material, processed once, one note written.
- **Date resolution:** if an email arrives the day after the meeting, use the meeting date (from transcript or body), not the email received date.
