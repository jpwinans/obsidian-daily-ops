# Alex Rivera Work PKB

Alex Rivera — Director of Platform Engineering at Acme Robotics. Reports to Sam Chen (VP Engineering). Start date: 2025-09-15. Scope: 14 engineers across two teams (Core Platform, Data Pipeline) plus shared ownership of the company-wide Observability initiative.

## Skills

Obsidian skills are installed at `.claude/skills/`. Use them for all file operations:
- **obsidian-markdown** — Obsidian-flavored markdown (.md): wiki-links, callouts, embeds, frontmatter
- **obsidian-bases** — Obsidian Bases (.base): native database views
- **json-canvas** — JSON Canvas (.canvas): visual node graphs

The daily-workflow skills (morning, prep, rollup, etc.) read the sections below.

## Vault Layout (ACE Framework)

Standard ACE layout — see `_Meta/Templates/` for note templates. No folder overrides.

## People

### Importance Tiers

| Tier | Weight | Role | Members |
|------|--------|------|---------|
| 1 | 5 | CEO / Skip-level | @Maya Patel |
| 2 | 4 | Direct manager | @Sam Chen |
| 3 | 3 | Peer leaders | @Dana Kim, @Jordan Reyes, @James Winans |
| 4 | 2 | Key stakeholders | @Priya Shah, @Marcus Webb |
| 5 | 1 | Direct reports | @Nia Okonkwo, @Tom Bauer |
| 6 | 0 | Anyone else | — |

### Direct Reports
- `@Nia Okonkwo` — Data Pipeline EM
- `@Tom Bauer` — Senior Staff Engineer (cross-team)

### Key Stakeholders
- `@Priya Shah` — Director of Product, owns the Observability roadmap with Alex
- `@Marcus Webb` — Director of SRE, partner on incident response
- `@James Winans` — peer Director of AI Product Engineering, partner on shared evaluation infra

## Channels (Slack)

### Daily Pulse
- `#team-platform` — main team channel
- `#team-data-pipeline` — Data Pipeline team
- `#proj-observability` — cross-functional Observability initiative

### Deploy / Alerts
- `#alerts-platform-deploys` — production deploy notifications
- `#alerts-pagerduty` — incident pages

### Leadership
- `#eng-directors` — peer-leader channel
- `#eng-leadership-slt` — SLT-only
- `#all-engineering`
- `#all-company`

## Projects / Initiatives

- **Observability Rollout** — company-wide tracing/metrics consolidation, Alex co-owns with Priya. Dashboard: `Efforts/Projects/Observability Rollout/Dashboard.md`
- **Pipeline Modernization** — multi-quarter migration off legacy ETL. Dashboard: `Efforts/Projects/Pipeline Modernization/Dashboard.md`
- **Platform SLO Initiative** — establishing SLOs for Core Platform services

## Gmail GTD Labels

- `📥 GTD/1 - Next Actions/@Email` — needs a reply
- `📥 GTD/1 - Next Actions/@Computer` — computer task (form, review, training)
- `📥 GTD/1 - Next Actions/@Agenda` — raise in a meeting
- `📥 GTD/1 - Next Actions/@Calls` — make a call
- `📥 GTD/2 - Waiting For` — ball in someone else's court
- `📥 GTD/3 - Projects` — multi-step outcome
- `📥 GTD/4 - Someday Maybe` — not urgent, review later
- `📥 GTD/5 - Reference` — useful info, no action
- `📥 GTD/6 - Delegated` — handed off, track completion

## Notion

- [Engineering Org Wiki](https://www.notion.so/example-org-wiki) — team pages, on-call rotations
- [Observability Design Doc](https://www.notion.so/example-obs-design) — current architecture
- [Sam ↔ Alex 1:1](https://www.notion.so/example-sam-alex-1on1) — running 1:1 notes

## Conventions

- **Daily note path:** `Calendar/Daily/YYYY-Mon/YYYY-MM-DD.md`
- **Weekly note path:** `Calendar/Weekly/YYYY-Www.md`
- **Person notes:** `People/@First Last.md`
- **Frontmatter required on every note:** `date`, `tags`, `status`, `related`
- **Status values:** `draft`, `active`, `review`, `complete`, `archived`
- **Tags:** `#meeting`, `#decision`, `#blocker`, `#idea`, `#adr`, `#1-1`
- **Task format:** `- [ ] Task @context #tag 📅 YYYY-MM-DD ⏫`
- **Task contexts:** `@meeting`, `@review`, `@blocked`, `@delegate`, `@discuss`
- **ADR numbering:** sequential (`Atlas/Engineering/ADR-001 Title.md`)
- **Meeting ignore patterns:** `Lunch`, `Focus Time`, `Block`, `OOO`, `Hold`
- **Stakeholder audiences:** `manager`, `team`, `leadership`

## Working Rules

- Knowledge base writes require approval; task organization handled automatically
- Add backlinks to relevant existing notes when creating or editing
- Create new topic/person notes when referenced entities don't exist
- Extract meeting action items as Tasks-plugin tasks with due dates
- Preserve existing frontmatter when editing; add fields, don't remove
- Daily notes go in `Calendar/Daily/YYYY-Mon/`, never at vault root

## Key Files

- `Home.md` — vault entry point and dashboard
- `Atlas/Leadership/Org Chart.md` — company org chart
- `People/@Alex Rivera.md` — Alex's own person note
- `Efforts/Projects/Observability Rollout/Dashboard.md` — primary project dashboard
