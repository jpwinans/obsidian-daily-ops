# {{Your Name}} Work PKB

{{One paragraph about you: name, role, manager, scope of responsibility. The skills read this for context.}}

## Skills

Obsidian skills are installed at `.claude/skills/`. Use them for all file operations:
- **obsidian-markdown** — Obsidian-flavored markdown (.md): wiki-links, callouts, embeds, frontmatter
- **obsidian-bases** — Obsidian Bases (.base): native database views
- **json-canvas** — JSON Canvas (.canvas): visual node graphs

The daily-workflow skills (morning, prep, rollup, etc.) read the sections below to know who's important to you, which channels to scan, and where things live.

## Vault Layout (ACE Framework)

```
+ Inbox/                → Quick capture, unprocessed items
Atlas/                  → Knowledge base
  Engineering/          → Systems design, technical knowledge
  Leadership/           → Org Chart
  MOCs/                 → Maps of Content
Calendar/               → Time-based notes
  Daily/                → Daily notes (YYYY-Mon/YYYY-MM-DD.md)
  Weekly/               → Weekly reviews (YYYY-Www.md)
  Meetings/             → Meeting notes
  1-1s/                 → 1:1 notes (one folder per person)
Efforts/                → Active work
  Projects/             → Current initiatives
  Areas/                → Ongoing responsibilities
  Outcomes/             → OKRs and goal tracking
People/                 → One note per person (@First Last.md)
  Teams/                → Team notes
Archive/                → Completed/reference materials
_Meta/                  → Templates, attachments
```

Override only if you've renamed folders.

## People

### Importance Tiers

Used by skills to weight tasks, emails, and Slack messages by who's behind them.

| Tier | Weight | Role | Members |
|------|--------|------|---------|
| 1 | 5 | Skip-level / CEO | TODO: @Name |
| 2 | 4 | Direct manager | TODO: @Name |
| 3 | 3 | Peer leaders | TODO: @Name, @Name |
| 4 | 2 | Key stakeholders | TODO: @Name, @Name |
| 5 | 1 | Direct reports | TODO: @Name |
| 6 | 0 | Anyone else | — |

### Direct Reports
- TODO: `@First Last` — team / focus area

### Key Stakeholders
- TODO: `@First Last` — role, why they matter

## Channels (Slack)

Used by `/morning:start`, `/morning:brief`, `/blocker:scan`. Channel names are enough; only set IDs for channels Slack search can't find by name (e.g. private channels you can't search across).

### Daily Pulse
- TODO: `#channel-name` — purpose

### Deploy / Alerts
- TODO: `#channel-name` (ID: `C0XXXXXX`) — only set ID if needed

### Leadership
- TODO: `#channel-name`

## Projects / Initiatives

Used by skills to cross-reference Slack/email mentions to active work.

- TODO: **Project Name** — one-line description, optional dashboard path (e.g., `Efforts/Projects/Project Name/Dashboard.md`)

## Gmail GTD Labels

Used by the email-triage portion of `/morning-start`. Skills use label *names*, not IDs — just match what your Gmail labels are actually called.

- `📥 GTD/1 - Next Actions/@Email` — needs a reply
- `📥 GTD/1 - Next Actions/@Computer` — computer task (form, review, training)
- `📥 GTD/1 - Next Actions/@Agenda` — raise in a meeting
- `📥 GTD/1 - Next Actions/@Calls` — make a call
- `📥 GTD/2 - Waiting For` — ball in someone else's court
- `📥 GTD/3 - Projects` — multi-step outcome
- `📥 GTD/4 - Someday Maybe` — not urgent, review later
- `📥 GTD/5 - Reference` — useful info, no action
- `📥 GTD/6 - Delegated` — handed off, track completion

If your label hierarchy differs, just edit the names above — the skills read whatever you write here.

## Notion (optional)

Used by `/morning:start`, `/prep:meeting`, `/prep:1on1` to cross-reference. Skip the section entirely if you don't use Notion.

- TODO: `[Page Title](url)` — what's there

## Conventions

- **Daily note path:** `Calendar/Daily/YYYY-Mon/YYYY-MM-DD.md` (e.g., `Calendar/Daily/2026-Apr/2026-04-30.md`)
- **Weekly note path:** `Calendar/Weekly/YYYY-Www.md`
- **Person notes:** `People/@First Last.md`
- **Team notes:** `People/Teams/Team Name.md` (no `@` prefix)
- **Frontmatter required on every note:** `date`, `tags`, `status`, `related`
- **Status values:** `draft`, `active`, `review`, `complete`, `archived`
- **Tags:** `#meeting`, `#decision`, `#blocker`, `#idea`, `#1-1`
- **Task format:**
  ```
  - [ ] Task description @context #tag 📅 YYYY-MM-DD
  - [ ] High priority task ⏫ 📅 YYYY-MM-DD
  - [ ] Medium priority task 🔼 📅 YYYY-MM-DD
  ```
- **Task contexts:** `@meeting`, `@review`, `@blocked`, `@delegate`, `@discuss`
- **Meeting ignore patterns:** `Lunch`, `Focus Time`, `Block`, `OOO` — skipped by `/prep:day`
- **Stakeholder audiences:** `manager`, `team`, `leadership` — used by `/stakeholder:update`

## Working Rules

- Knowledge base writes (new notes, edits to existing content): require approval
- Task organization (tagging, moving, prioritizing): handle automatically
- Always add backlinks to relevant existing notes when creating or editing
- Create new topic/person notes when referenced entities don't exist yet
- When processing meeting notes, extract action items as Tasks-plugin tasks
- Preserve existing frontmatter when editing notes; add fields, don't remove
- Daily notes go in `Calendar/Daily/YYYY-Mon/`, never at vault root

## Key Files

- `Home.md` — Vault entry point and dashboard (vault root)
- `Atlas/Leadership/Org Chart.md` — Your org chart
- `Atlas/MOCs/` — Maps of Content
- `People/@{{Your Name}}.md` — Your person note (role, reports, scope, stakeholders)
- `_Meta/Templates/` — All note templates
