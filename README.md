# Obsidian Daily Ops

A self-contained Obsidian vault and Claude Code skill pack for running your day: morning briefing, meeting prep, daily/weekly rollups, action-item triage, blocker scanning, and email triage — wired to your Slack, Gmail, Google Calendar, and Notion accounts.

**License:** MIT

---

## Quick start

1. **Clone the repo** to wherever you keep your Obsidian vaults.
   ```bash
   git clone https://github.com/jpwinans/obsidian-daily-ops.git <vault-name>
   cd <vault-name>
   ```

2. **Open the folder in Obsidian** as a vault. The folder tree is already set up.

3. **Install Claude Code** if you don't have it: <https://docs.claude.com/code>.

4. **(optionally) Open Claude Code in the vault directory** and run **`/setup-vault`**.

   The wizard walks you through every question needed to configure the vault — your name and role, the people who matter (CEO, manager, peers, direct reports, stakeholders), Slack channels you watch, active projects, Gmail labels, optional Notion pages. ~5–10 minutes of plain-English Q&A. It writes your `CLAUDE.md` for you at the end after showing you a preview.

   **Prefer to edit by hand?** Copy `CLAUDE.md.template.md` → `CLAUDE.md` and fill in the `TODO` markers. See `CLAUDE.md.example.md` for a filled-out reference.

   *Why the double `.md` extension?* Obsidian only renders files whose name ends in `.md`. Naming the source files `CLAUDE.md.template.md` and `CLAUDE.md.example.md` lets you read them inside Obsidian. Claude Code reads `CLAUDE.md` directly — the `.template.md` / `.example.md` files are for human reference only.

5. **Connect MCP servers** for the integrations you want. The skills work with whatever subset you have — they degrade gracefully when an MCP isn't connected.
   - **Slack** — used by `/morning-start`, `/morning-brief`, `/blocker-scan`
   - **Gmail** — used by `/morning-start` (email triage section) and `/meeting-ingest`
   - **Google Calendar** — used by `/prep-day`
   - **Google Drive** — used by `/meeting-ingest` for transcript files
   - **Notion** (optional) — used by `/prep-meeting`, `/prep-1on1`, `/morning-start`

6. **Copy the permissions template:**
   ```bash
   cp .claude/settings.local.json.example .claude/settings.local.json
   ```
   This pre-approves every MCP tool the skills use. Trim what you don't need.

7. **Try `/morning-start`** for your first daily briefing.

### Cleanup after you understand the vault

The template ships with sample notes so you can see the shape of a working vault before you have your own data. Once you've poked around and understood the structure, **delete these — they're scaffolding, not your actual content**:

- `Calendar/Daily/2026-Apr/2026-04-30.md` — sample daily note showing what `/morning-start` produces
- `Calendar/1-1s/James Winans/2026-04-30.md` — sample peer 1:1 note (and the parent `Calendar/1-1s/James Winans/` folder once empty)
- `Calendar/Meetings/Eng Directors Sync/2026-04-30.md` — sample recurring-meeting note (and the parent folder once empty)
- `People/@James Winans.md` — sample peer-leader person note
- `People/@Example Person.md` — bare-schema example person note showing the expected frontmatter
- `Action Items.md` — sample seed file. Replace by running `/action-items-compress` once your vault has accumulated real tasks across daily notes / 1:1s / meetings.
- `Atlas/Leadership/Org Chart.md`, `Atlas/MOCs/{Engineering,Leadership,Product} MOC.md`, `Efforts/Outcomes/Current OKRs.md`, `Home.md` — these are placeholders showing common structures. Keep, replace, or delete as fits your workflow.

The two CLAUDE files (`CLAUDE.md.template.md` and `CLAUDE.md.example.md`) are reference material — keep them around if you ever need to re-bootstrap or share the structure with someone else.

---

## Your daily flow at a glance

Once you're set up, the whole rig runs on **two commands**:

```
☀️  Start of day:   /morning-start    ← briefing + meeting prep, all in one
🌙  End of day:     /rollup-daily     ← ingest meeting transcripts + reconcile
```

`/morning-start` auto-chains into `/prep-day` (calendar read + per-meeting prep notes). `/rollup-daily` auto-chains into `/meeting-ingest` (pulls Gemini Notes / Google Meet transcripts from Gmail and enriches the day's meeting notes). You don't run those four sub-skills directly.

**Friday end of day:** add `/rollup-weekly` after `/rollup-daily` to close out the week.

That's the whole loop. The other 16+ skills are opportunistic — see the **Recommended daily routine** section below for when to reach for them.

---

## What's in the box

### The vault (ACE framework)

```
+ Inbox/        Quick capture
Atlas/          Knowledge base (Engineering, Leadership, MOCs)
Calendar/       Daily, Weekly, Meetings, 1-1s
Efforts/        Projects, Areas, Outcomes
People/         One note per person + Teams/
Archive/        Completed materials
_Meta/          Templates
```

### The skills (`.claude/skills/`)

**User-invocable workflow skills:**

| Slash | What it does |
|-------|--------------|
| `/morning-start` | Full daily briefing — Slack + Gmail + vault state + risk digest, written to today's daily note |
| `/morning-brief` | Lighter version — vault state and overnight Slack only |
| `/prep-day` | Reads Google Calendar, classifies meetings, runs prep skills per meeting |
| `/prep-meeting` | Pre-populates a meeting note with context from vault, Slack, Notion |
| `/prep-1on1` | 1:1 prep — recent activity, last 1:1 follow-ups, Slack mentions |
| `/rollup-daily` | End-of-day reconciliation against the morning plan |
| `/rollup-weekly` | Weekly review — accomplishments, in-progress, blockers, next week focus |
| `/meeting-ingest` | Pulls Gemini / Google Meet transcript emails and enriches vault meeting notes |
| `/action-items-scan` | Buckets open tasks by age (stale / aging / due today / horizon) |
| `/action-items-compress` | Consolidates duplicate action items across the vault |
| `/action-items-triage` | Interactive HTML triage view (Prune / Focus / Balance) |
| `/blocker-scan` | Risk digest — Slack + vault, classified by ownership and age |
| `/setup-vault` | Interactive wizard for first-time CLAUDE.md configuration (run once) |
| `/refresh-vault` | Detects information drift and DRY violations across the vault |
| `/hyper-explore-vault` | Multi-agent vault audit (structural health, gaps, hidden threads) |

**Helper skills (auto-loaded by the workflow skills above; hidden from the slash menu):**

- `vault-config` — parses your CLAUDE.md schema
- `slack-triage` — ownership classification + age tiering
- `gmail-classifier` — inbox classification by ownership + sender importance
- `meeting-notes` — meeting note format + transcript extraction

**Generic Obsidian-format skills:**

- `obsidian-markdown` — wikilinks, callouts, embeds, frontmatter
- `obsidian-bases` — `.base` database views
- `json-canvas` — `.canvas` visual graphs

---

## How customization works

Every workflow skill reads your `CLAUDE.md`. The `vault-config` helper parses these documented sections:

- `## Vault Layout` — folder paths (only override if you renamed something)
- `## People` — importance tiers, direct reports, key stakeholders
- `## Channels (Slack)` — daily-pulse, deploy/alerts, leadership channels
- `## Projects / Initiatives` — active work to cross-reference
- `## Gmail Labels` — your label hierarchy by name
- `## Notion` — optional reference page list
- `## Conventions` — daily-note path, frontmatter, task format, ignore patterns

**No second config file.** Everything lives in `CLAUDE.md`. Skills degrade gracefully when a section is missing — `/morning-start` with an empty `## Channels` skips the Slack scan and tells you what to add.

---

## Recommended daily routine

The skills are designed for a two-touch day: one command in the morning, one at end of day. Most everything else slots in opportunistically.

### Daily

| When | Command | What you get |
|------|---------|--------------|
| **Start of day** | `/morning-start` | Scans Slack (24h on weekdays / 72h on Mondays), Gmail inbox, vault state. Writes today's daily note with risk digest, ownership-classified items, top-3 priorities, deploy status, email triage. **Auto-chains into `/prep-day`** which reads your calendar, classifies each meeting (1-1 / recurring / ad-hoc / ignore), and runs prep skills per event. By the time it finishes, your daily note + every meeting note is ready. |
| **End of day** | `/rollup-daily` | Auto-runs `/meeting-ingest` first (pulls Gemini Notes / Google Meet transcript emails from Gmail and enriches the corresponding vault meeting notes — synthesized, not raw). Then reconciles morning plan vs reality across every section of today's daily note (tasks done/deferred/moot, focus achieved, meetings logged, EOD summary). |

### How meeting notes get written

Meeting notes are written automatically by `/meeting-ingest`, which runs as part of `/rollup-daily` at end of day. It searches Gmail for Gemini Notes / Google Meet transcript emails from the day, then synthesizes each into a structured meeting note in `Calendar/Meetings/<Name>/YYYY-MM-DD.md` (or `Calendar/1-1s/<Person>/YYYY-MM-DD.md` for 1:1s). You don't run anything directly.

If a meeting wasn't recorded by Gemini or Google Meet (in-person, untranscribed call, etc.), no transcript reaches Gmail and no note gets generated automatically — you'll need to write that one by hand from your live notes.

### Weekly

| When | Command | What you get |
|------|---------|--------------|
| **Monday morning** (after `/morning-start`) | `/action-items-triage` | Interactive HTML page for prune / focus / balance. Pick 3–5 rocks for the week. |
| **Friday end of day** | `/rollup-daily` then `/rollup-weekly` | Closes today, then aggregates the whole week's activity into a structured retrospective. |

### Advanced usage

Skills outside the daily two-touch flow. Run on cadences that match what each one actually does — none of these belong in your daily routine.

#### Action-items pipeline — three skills, one rhythm

| Cadence | Skill | What it does |
|---------|-------|--------------|
| Once a week | `/action-items-compress` | Build the consolidated backlog |
| Daily (cheap) | `/action-items-scan` | Quick "what's stale, what's due today" check |
| Monday + ad hoc | `/action-items-triage` | Interactive prune / focus / balance ritual |

**`/action-items-compress`** — heavy. Walks the entire vault, deduplicates every `- [ ]` task across daily notes / 1:1s / meeting notes / project notes / etc., scores each by priority (manager weight + commitment + deadline + people affected + blocker status), and writes the consolidated ranked backlog to `Action Items.md` at vault root. Demotes source tasks to breadcrumbs (`→ [[Action Items]]`) so there's a single source of truth. **Asks before overwriting** — review the proposed list before approving. Run weekly (Sunday night or Monday morning) or before a major triage session.

**`/action-items-scan`** — cheap, runs on Haiku, ~30-second orientation. Reads `Action Items.md`, buckets open tasks by age (🔴 stale > 3 days overdue / 🟠 aging 1–3 days / 🟡 due today / 🟢 due this week / ℹ️ on the horizon). Updates only the scan-timestamp header in the file; output goes to stdout. Use whenever you want a quick read on what's slipping.

**`/action-items-triage`** — interactive HTML page generated to vault root. Three tabs: **Triage** (per-task keep / defer / delegate / kill / complete decisions, keyboard-driven so it's fast), **Focus** (pick 3–5 rocks for `🎯 This Week`), **Balance** (portfolio view across Strategic / Operational / Relational work). Make decisions in the browser, then come back to Claude and say "apply decisions" — it writes the changes back to `Action Items.md` (defers update due dates, kills move to Completed with `#cancelled`, delegates add `@delegate [[@Person]]` and a follow-up note, focus repopulates the `🎯 This Week` section).

The pipeline assumes `Action Items.md` exists. The seed file at vault root has the canonical structure — replace it with your real backlog by running `/action-items-compress` for the first time once your vault has accumulated tasks across a few weeks of notes.

#### `/refresh-vault` — monthly drift + DRY auditor

Reads recently-modified files (last 24h = "Tier 1 truth") and audits older notes against them for contradictions, stale facts, dead links, and content duplicated across multiple notes instead of linked. Two classes of fixes:

- **Mechanical** — value canonically asserted in a Tier 1/2 source. Applied directly (date corrections, frontmatter updates, dead-link removals, replacing references to archived notes with their `superseded-by` target).
- **Editorial** — requires composing prose or making judgment calls. Applied with stricter verification — re-reads the cited source after editing and reverts the change if the prose isn't backed by canonical assertions.

Heavy (`model: opus`, `effort: max`). Don't run daily — monthly cadence is right unless you've made a major restructuring. Produces a changelog at `Calendar/Daily/YYYY-Mon/refresh-vault-YYYY-MM-DD.md` listing every fix applied (split by mechanical / editorial), every reverted edit with the verification failure, and any "Needs Human Decision" findings — items requiring judgment beyond the editorial bar (e.g., resolving a contradiction between two equally-fresh sources).

Skipped on fresh clones (under 8 .md files). Becomes useful once you have ~20+ interconnected notes.

#### `/hyper-explore-vault` — quarterly deep audit

The heaviest skill in the suite. Launches 6 background agents in parallel (one per region: Atlas / Calendar / People / Efforts / Root + Templates / Wikilink graph), each reading every file in its scope. After several minutes, returns a synthesis report covering:

- **Structural health** — vault score, dead links, orphan notes, entities referenced but missing their own notes, empty/stub notes
- **The meaning** — what is this vault actually about? what story does the timeline tell? where is attention going? what does it imply you're worried about?
- **Hidden threads** — implicit connections across multiple notes that aren't explicitly linked but should be (the "aha" insights that only emerge from reading everything at once)
- **Strategic contradictions** — places where different notes imply different priorities
- **What's missing** — person notes that should exist, topics referenced but never given their own note, process gaps
- **Priority queue** — highest-leverage actions ranked by impact

Run when planning a quarter, before a strategic shift, or when the vault feels sprawling and you want a third-party read on it. Optionally offers to create a Hidden Threads MOC, stub person notes for missing people, dead-link fix instructions, or a Vault Health canvas after delivering the synthesis. Falls back to sequential execution on small vaults (under ~50 files) or if the harness disallows parallel agents.

### The 80% case

```
Morning:        /morning-start            ← briefing + meeting prep, all in one
End of day:     /rollup-daily             ← ingest transcripts + reconcile
Friday EOD:     /rollup-daily, /rollup-weekly
```

That's the whole skeleton. Two commands a day. The auto-chaining does the rest:
`/morning-start` → `/prep-day` → `/prep-meeting` / `/prep-1on1` (per event)
`/rollup-daily` → `/meeting-ingest` → `/prep-meeting` / `/prep-1on1` (if a transcript arrived for a meeting that didn't have a prep note yet)

Everything else slots in when you need it — most days you don't.

---

## Troubleshooting

**`/morning-start` says "no channels configured"** — Add a `## Channels` section to your CLAUDE.md with at least one `#channel-name` bullet.

**Slack search isn't finding a channel by name** — That channel probably needs an explicit ID. Add it as `#channel-name (ID: C0XXXXXX)` in CLAUDE.md. To find a channel ID: open it in the Slack desktop app, click the channel name → Copy ID.

**Gmail labels don't match** — The skills use label *names*, not IDs. Whatever you write in `## Gmail Labels` is what they look for. If your labels are nested differently (e.g., `Inbox/Action`), just write that.

**Skills don't appear in the slash menu** — Make sure you launched Claude Code from the vault directory (the one with `.claude/` in it). Helper skills like `vault-config` are intentionally hidden from the menu.

**Daily note ends up in the wrong folder** — Override `## Conventions` → "Daily note path" in CLAUDE.md.

---

## Customizing further

- **Add a skill** — drop a directory at `.claude/skills/<skill-name>/` containing `SKILL.md` with frontmatter (`name`, `description`, `model`, `allowed-tools`). It auto-appears as `/skill-name` in the slash menu. Note: Claude Code only discovers skills one level deep — nested directories like `.claude/skills/group/name/SKILL.md` are NOT loaded. Use hyphens instead: `.claude/skills/group-name/SKILL.md`.
- **Change a skill's model** — edit the `model:` line in that skill's `SKILL.md` (e.g., `claude-haiku-4-5` for cheaper, `claude-opus-4-7` with `effort: max` for hardest tasks).
- **Restrict a skill's tools** — edit `allowed-tools:` in its frontmatter.
- **Hide a skill from the slash menu** — set `user-invocable: false` in frontmatter.

See [Claude Code Skills docs](https://code.claude.com/docs/en/skills.md) for the full frontmatter reference.

---

## Obsidian plugins

The templates and skills in this vault assume a few Obsidian community plugins. They ship in `.obsidian/plugins/` but **Obsidian won't auto-enable them** on first open — you'll see a "Trust author and enable plugins?" prompt or have to enable each one manually under **Settings → Community plugins → Installed plugins**.

| Plugin | What the template uses it for |
|--------|-------------------------------|
| **Dataview** | The Daily Note's "Overdue Tasks" / "Due Today" / "Stale Relationships" / "Inbox" widgets, the Engineering MOC's queries, the Project template's "Decisions" pull, person-note "Meeting History" tables, and the Action Items file's task queries. **Required** — most templates render empty without it. |
| **Tasks** | Task syntax with priority emojis (⏫ 🔼), due dates (`📅 YYYY-MM-DD`), completion (✅), context tags (`@meeting`, `@discuss`), plus the Weekly Note's `tasks` query block. **Required** if you want the action-items pipeline to work. |
| **Calendar** | Sidebar for navigating Daily / Weekly notes by date. Optional but strongly recommended — most users expect it for `Calendar/Daily/` navigation. |
| **Templater** | Substitutes `{{date:YYYY-MM-DD}}` / `{{title}}` placeholders in `_Meta/Templates/` when you create a new note from a template. Optional — without it, those placeholders stay literal and you fill them in manually. |
| **Auto Link Title** | Fetches the page title when you paste a URL into a note (so `https://...` becomes `[Page Title](...)`). Optional, quality-of-life. |
| **Editing Toolbar** | Floating formatting bar in the editor. Optional, UI preference. |

**On first open:**

1. Obsidian shows the "Trust author and enable plugins?" dialog. Click **Trust author and enable plugins** if you trust this template's source.
2. If you skipped the trust dialog: open **Settings → Community plugins**, toggle on **Restricted mode → off**, then enable each plugin under **Installed plugins**.
3. If a plugin is missing from `.obsidian/plugins/` (e.g., your clone is incomplete), install via **Browse community plugins** — they're all on the official Obsidian registry.

If you want a leaner vault, you can remove plugins you don't use — but expect Dataview-driven widgets in the Daily Note and MOCs to render as empty code blocks until you re-enable Dataview.

---

## License

MIT — see [LICENSE](LICENSE).

## Contributing

Issues and pull requests welcome.
