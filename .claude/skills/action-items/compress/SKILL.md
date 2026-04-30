---
name: action-items-compress
description: |
  Scans the entire vault for action items, deduplicates them, scores each
  by priority (PersonWeight × 100 + Committed × 80 + ...), writes one
  consolidated `📋 Action Items.md` at vault root, and demotes all source
  tasks to breadcrumbs. Asks before writing. Triggers: "compress action
  items", "consolidate my tasks", "/action-items compress".
model: claude-opus-4-7
effort: high
allowed-tools: Read Write Edit Glob Grep
---

# /action-items:compress — Consolidate vault tasks into a single ranked backlog

Scan the vault for all action items, deduplicate, score, write a single `📋 Action Items.md` at vault root, and demote source tasks to breadcrumbs.

## Context

Action items scatter across daily notes, meeting notes, 1:1s, project plans, and roadmaps. The same task often appears in multiple files. This skill automates consolidation: find all tasks, dedupe, score, sort, leave breadcrumbs in source files.

## Instructions

### Step 1 — Read context

1. Load `vault-config` for `vault.layout`, `personTiers`, `conventions`.
2. Load `obsidian-markdown` for formatting.
3. If `📋 Action Items.md` already exists at vault root, read it and ask the user before overwriting.

### Step 2 — Build a complete file manifest

**Do not rely on a single vault-wide grep — large vaults may truncate results and silently miss files.**

Use Glob to enumerate every `.md` file in each included directory (paths from `vault-config.vault.layout`):

- `<vault.layout.daily>/**/*.md`
- `<vault.layout.oneOnOnes>/**/*.md`
- `<vault.layout.meetings>/**/*.md`
- `<vault.layout.projects>/**/*.md` (and other Efforts/ subdirs)
- `Atlas/**/*.md`
- `<vault.layout.people>/**/*.md`
- Vault root: `Home.md` (exclude `📋 Action Items.md` itself)

**Exclude:** `.claude/`, `.trash/`, `.obsidian/`, `<vault.layout.templates>/`, `<vault.layout.archive>/`, `📋 Action Items.md` itself.

State the count: "Found N files to scan." Proceed only when the manifest is complete.

Now grep each directory for `- [ ]` (open) and `- [x]` (completed). Cross-check against the manifest.

For each task captured: full line text, source file path, line number, sub-items (indented lines beneath) as context.

### Step 3 — Deduplicate

Read source context (±5 lines) around each task to understand what it relates to.

Identify tasks pointing to the **same unit of work** by matching:
- Identical or near-identical task text (after stripping emoji, dates, context tags)
- Same person + same topic
- Same project/initiative reference

For each duplicate group:
- Keep the most descriptive version as canonical
- Preserve the earliest due date
- Preserve the highest priority emoji
- Collect all source paths for inline backlinks

Assign each deduplicated task a category tag. Default categories:

| Tag | Covers |
|-----|--------|
| `#strategic` | Architecture, frameworks, direction-setting |
| `#operational` | Execution, reviews, approvals, coordination |
| `#relational` | 1:1 follow-ups, intros, relationship building |
| `#admin` | Direct reports, HR, scorecards, 1:1 cadences |
| `#reading` | Reading backlog, docs review, research |
| `#blocked` | Tasks tagged `@blocked` |

If the user has additional category tags documented in CLAUDE.md `## Conventions` (e.g., per-project tags), include those too.

### Step 4 — Score every task

```
Priority = (PersonWeight × 100)
         + (Committed × 80)
         + (DueDate × 70)
         + (PeopleAffected × 60)
         + (Severity × 50)
         + (IsTactic × 40)
         + (IsStrategy × 30)
```

| Factor | Scale | Inference |
|--------|-------|-----------|
| **PersonWeight** | 0–5 | Highest-tier person *driving* the work, not just mentioned. Trace the source chain: if a task came out of one person's 1:1 but exists because a higher-tier person mandated it, use the higher tier. From `vault-config.people.tiers`. Ask: "Who would be most disappointed if this wasn't done?" |
| **Committed** | 0/1 | `1` if the task appears in a 1:1 or meeting Action Items section, or the user explicitly agreed. `0` if self-generated from reading or aspirational planning. |
| **DueDate** | 0–5 | Relative to today. Overdue >3 days = 5. Overdue 1–3 = 4. Today = 3. This week = 2. This month = 1. None or >30 days = 0. |
| **PeopleAffected** | 0–5 | Count distinct `[[@Person]]` mentions in task + sub-items. Plus +1 if task appears in 3+ source files. Cap 5. |
| **Severity** | 0–5 | `5` if blocking another open task. `3` if it references a bug ticket or risk/incident. `0` for normal. Tasks themselves `@blocked` get `0`. |
| **IsTactic** | 0/1 | `1` if concrete and near-term: action verb (meet, review, send, ping, schedule) + specific person/artifact. |
| **IsStrategy** | 0/1 | `1` if links to MOCs, `Efforts/Areas/`, or describes framework/process building with due >2 weeks out. Mutually exclusive with IsTactic — prefer IsTactic when both apply. Default IsTactic = 1 if neither applies. |

**Tie-breaking:** same score → earliest due date → alphabetical.

### Step 5 — Ask before writing

Present the full sorted list to the user:

> "I found N open and M completed tasks across P files. After deduplication: X unique open, Y waiting, Z completed.
>
> Top 10 by score:
> 1. [score] Task #tag 📅 date
> 2. ...
>
> Should I:
> 1. **Proceed** — write `📋 Action Items.md` and demote all source tasks
> 2. **Adjust** — tell me what to change
> 3. **Skip** — no changes"

Only proceed to Steps 6–8 after user approval.

### Step 6 — Build `📋 Action Items.md`

```markdown
---
date: YYYY-MM-DD
tags:
  - action-items
status: active
---
# Action Items — YYYY-MM-DD

> [!info] Auto-generated by `/action-items:compress`. Open items stratified by work type (Strategic / Operational / Relational) and sorted by priority score within each section. Categories are inline #tags. Source tasks demoted with `→ [[📋 Action Items]]` breadcrumb.

---

## 🎯 This Week
*(Max 5 items — populate manually each Monday from Open. Refresh weekly.)*



---

## Open

### Strategic
*(Architecture decisions, org direction, measurement, security frameworks — requires protected deep-work time)*

- [ ] Task description ⏫ #tag 📅 YYYY-MM-DD → [[Source Note]]

### Operational
*(Execute, review, coordinate, approve — can be batched or delegated)*

- [ ] Task description ⏫ #tag 📅 YYYY-MM-DD → [[Source Note]]

### Relational
*(1:1 follow-ups, connections, introductions — neglect becomes visible fast)*

- [ ] Task description 🔼 #tag 📅 YYYY-MM-DD → [[Source Note]]

---

## Waiting On Others
*(Ball is in someone else's court. Scan weekly — escalate on or before trigger date.)*

- [ ] [[@Person]]: Task description ⏫ #tag 📅 YYYY-MM-DD → [[Source Note]]
  ⚡ escalate by: YYYY-MM-DD — [escalation note]

---

## Someday / Maybe
*(Not this month. Review-by date noted. Activate or drop at weekly review — if it survives three reviews untouched, drop it.)*



---

## Completed

> [!faq]- Completed Items → [[Archive/Completed Action Items]]
> All completed and moot tasks have been archived. See [[Archive/Completed Action Items]] for the full history organized by theme and batch.

- [x] Task description 📅 YYYY-MM-DD ✅ YYYY-MM-DD #tag → [[Source Note]]
```

**Priority emoji** (based on score):
- `⏫` — score ≥ 700
- `🔼` — score 400–699
- *(none)* — score < 400

**Work-type classification** for Open tasks:

| Work Type | Assign when |
|-----------|-------------|
| **Strategic** | Decision, framework, or direction-setting output. Tags like `#strategic` or domain-specific architecture/security tags. |
| **Relational** | About a human relationship: 1:1 follow-up, intro, connection. Tags `#relational`, `#people`. Verbs: schedule, connect, meet, reach out. |
| **Operational** | Everything else: reviews, approvals, status tracking, unblocking. |

When a task fits multiple, prefer highest-leverage (Strategic > Relational > Operational). Sort by score desc within each section.

**Source backlinks:** when a task came from multiple files, list all comma-separated: `→ [[Source 1]], [[Source 2]]`.

**Waiting On Others format:** `[[@Person]]: task description` + `⚡ escalate by: YYYY-MM-DD — [one-line note]` on the next indented line. Infer a reasonable escalation date (3–7 days from now if none) and note from context.

**Someday / Maybe:** tasks scored < 300 with no due date within 14 days, or aspirational/reading/exploration with no committed deadline. Add `📅 review: YYYY-MM-DD` (4 weeks out) so they surface at the next monthly review.

**Moot tasks** (person departed, initiative cancelled, superseded): omit from Open, add a brief entry at the bottom of Completed with a note.

### Step 7 — Demote source tasks

**Goal:** zero remaining `- [ ]` or `- [x]` checkboxes in any in-scope source file. Every file in your manifest must be processed — not just files that contributed tasks.

Process in directory batches. For each batch, edit every file, then verify before moving on.

**Batch order:**
1. `<vault.layout.daily>/`
2. `<vault.layout.oneOnOnes>/`
3. `<vault.layout.meetings>/`
4. `Efforts/`, `Atlas/`, `<vault.layout.people>/`, vault root

**For each file:**
1. Read to confirm current state
2. Every `- [ ]` line: convert to `-` and append ` → [[Action Items]]`
3. Every `- [x]` line: convert to `-` and append ` → [[Action Items]]`
4. Write the edit
5. If a task line is inside a callout (`> `), preserve the `> ` prefix when demoting

**After each batch:** grep that directory for remaining `- [ ]` to confirm zero remain before starting the next batch.

**Do NOT demote in:**
- `📋 Action Items.md` itself
- `<vault.layout.templates>/` files

**Editing rules:**
- Preserve all frontmatter — only change task lines
- Preserve indentation and surrounding content
- Preserve all other content on the line (priority emoji, dates, descriptions, existing wikilinks)

### Step 8 — Hard verification gate

**Do not proceed to Step 9 until this passes.**

Grep ALL in-scope directories for `- [ ]`:
- Search: `- [ ]`
- Paths: `<vault.layout.daily>`, `Efforts/`, `Atlas/`, `<vault.layout.people>/`, `Home.md`
- Exclude: `📋 Action Items.md`, `<vault.layout.templates>/`, `.claude/`, `<vault.layout.archive>/`

Zero matches → proceed. Any matches → read those files, demote, re-grep. Repeat until zero.

Also grep for `- [x]` and demote any remaining.

### Step 9 — Present summary

- Total tasks found (open + completed)
- Unique tasks after deduplication
- Score range (highest → lowest)
- Number of source files modified
- Demoted task count
- Top 5 by score (sanity check)
- Confirmation: "Grep for `- [ ]` in <scope> returned 0 results."

## Important rules

- **Always ask before writing** — Step 5 presents the plan first.
- **Manifest before grep** — enumerate with Glob first, then grep. Never assume a single grep is exhaustive.
- **Demote everything** — every `- [ ]` in every in-scope file becomes `-` with a breadcrumb, regardless of whether it ended up in Action Items.md.
- **Batch and verify** — process in directory batches; grep each before moving on.
- **Zero tolerance** — Step 8 is a hard gate.
- **Preserve frontmatter** — only change task lines.
- **One task, one entry** — same task in 5 files becomes ONE task with all 5 sources listed.
- **PersonWeight traces to ultimate authority** — who would be most affected by non-completion, not just who's mentioned.
