# obsidian-daily-ops

A self-contained Obsidian vault and Claude Code skill pack for running your day: morning briefing, meeting prep, daily/weekly rollups, action-item triage, blocker scanning, and email triage — wired to your Slack, Gmail, Google Calendar, and Notion accounts.

**License:** MIT

---

## Quick start

1. **Clone the repo** to wherever you keep your Obsidian vaults.
   ```bash
   git clone https://github.com/<you>/obsidian-daily-ops.git my-vault
   cd my-vault
   ```

2. **Open the folder in Obsidian** as a vault. The folder tree is already set up.

3. **Fill in `CLAUDE.md`.** Copy the template and edit:
   ```bash
   cp CLAUDE.md.template CLAUDE.md
   ```
   Replace every `TODO` with your real people, channels, projects, and Gmail labels. See `CLAUDE.md.example` for a fully filled-out reference.

4. **Install Claude Code** if you don't have it: <https://docs.claude.com/code>.

5. **Connect MCP servers** for the integrations you want. The skills work with whatever subset you have — they degrade gracefully when an MCP isn't connected.
   - **Slack** — used by `/morning-start`, `/morning-brief`, `/blocker-scan`
   - **Gmail** — used by `/morning-start`, `/gmail-triage`, `/meeting-ingest`
   - **Google Calendar** — used by `/prep-day`
   - **Google Drive** — used by `/meeting-ingest` for transcript files
   - **Notion** (optional) — used by `/prep-meeting`, `/prep-1on1`, `/morning-start`

6. **Copy the permissions template:**
   ```bash
   cp .claude/settings.local.json.example .claude/settings.local.json
   ```
   Add MCP tool patterns you want pre-approved (see comment in the file).

7. **Open Claude Code in the vault directory** and try `/morning-start`.

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
| `/gmail-triage` | Classifies inbox into your GTD label structure |
| `/meeting-extract` | Restructures a raw meeting note into the template format |
| `/meeting-ingest` | Pulls Gemini / Google Meet transcript emails and enriches vault meeting notes |
| `/action-items-scan` | Buckets open tasks by age (stale / aging / due today / horizon) |
| `/action-items-compress` | Consolidates duplicate action items across the vault |
| `/action-items-triage` | Interactive HTML triage view (Prune / Focus / Balance) |
| `/blocker-scan` | Risk digest — Slack + vault, classified by ownership and age |
| `/stakeholder-update` | Drafts audience-specific status updates |
| `/decision` | Creates a numbered ADR in `Atlas/Engineering/` |
| `/refresh-vault` | Detects information drift and DRY violations across the vault |
| `/hyper-explore-vault` | Multi-agent vault audit (structural health, gaps, hidden threads) |

**Helper skills (auto-loaded by the workflow skills above; hidden from the slash menu):**

- `vault-config` — parses your CLAUDE.md schema
- `slack-triage` — ownership classification + age tiering
- `gmail-gtd-triage` — GTD label classification
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
- `## Gmail GTD Labels` — your label hierarchy by name
- `## Notion` — optional reference page list
- `## Conventions` — daily-note path, frontmatter, task format, ignore patterns

**No second config file.** Everything lives in `CLAUDE.md`. Skills degrade gracefully when a section is missing — `/morning-start` with an empty `## Channels` skips the Slack scan and tells you what to add.

---

## Troubleshooting

**`/morning-start` says "no channels configured"** — Add a `## Channels` section to your CLAUDE.md with at least one `#channel-name` bullet.

**Slack search isn't finding a channel by name** — That channel probably needs an explicit ID. Add it as `#channel-name (ID: C0XXXXXX)` in CLAUDE.md. To find a channel ID: open it in the Slack desktop app, click the channel name → Copy ID.

**Gmail labels don't match** — The skills use label *names*, not IDs. Whatever you write in `## Gmail GTD Labels` is what they look for. If your labels are nested differently (e.g., `Inbox/Action`), just write that.

**Skills don't appear in the slash menu** — Make sure you launched Claude Code from the vault directory (the one with `.claude/` in it). Helper skills like `vault-config` are intentionally hidden from the menu.

**Daily note ends up in the wrong folder** — Override `## Conventions` → "Daily note path" in CLAUDE.md.

**`/decision` puts ADRs somewhere weird** — Override `## Vault Layout` to point at your preferred ADR folder.

---

## Customizing further

- **Add a skill** — drop a directory at `.claude/skills/<skill-name>/` containing `SKILL.md` with frontmatter (`name`, `description`, `model`, `allowed-tools`). It auto-appears as `/skill-name` in the slash menu. Note: Claude Code only discovers skills one level deep — nested directories like `.claude/skills/group/name/SKILL.md` are NOT loaded. Use hyphens instead: `.claude/skills/group-name/SKILL.md`.
- **Change a skill's model** — edit the `model:` line in that skill's `SKILL.md` (e.g., `claude-haiku-4-5` for cheaper, `claude-opus-4-7` with `effort: max` for hardest tasks).
- **Restrict a skill's tools** — edit `allowed-tools:` in its frontmatter.
- **Hide a skill from the slash menu** — set `user-invocable: false` in frontmatter.

See [Claude Code Skills docs](https://code.claude.com/docs/en/skills.md) for the full frontmatter reference.

---

## License

MIT — see [LICENSE](LICENSE).

## Contributing

Issues and pull requests welcome.
