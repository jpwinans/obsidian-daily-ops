---
name: vault-config
description: |
  Parses the user's CLAUDE.md schema (## People, ## Channels, ## Projects,
  ## Gmail GTD Labels, ## Vault Layout, ## Conventions) into a structured
  config object that other skills consume. Auto-loaded by every workflow
  skill in the suite. Returns sensible defaults for missing sections.
user-invocable: false
model: inherit
allowed-tools: Read
---

# vault-config — CLAUDE.md schema parser

You are a configuration parser. Other skills load you when they need to know who's important to the user, which Slack channels to scan, which GTD labels exist, or where notes live in the vault.

## When to use this skill

Other workflow skills invoke you near the start of their execution. They will say things like "load vault-config" or "parse CLAUDE.md per the vault-config skill." Do not run on your own — wait to be invoked.

## What you do

1. **Read** `CLAUDE.md` from the vault root.
2. **Parse** the documented sections below.
3. **Validate** required ones are present.
4. **Return** a structured object the calling skill can use.

If `CLAUDE.md` does not exist, return `{ status: "missing", message: "No CLAUDE.md found. Copy CLAUDE.md.template to CLAUDE.md and fill it in." }` and stop.

## Schema you parse

### `## Vault Layout`
Optional. Free-form prose describing folder layout. If overrides are documented here, capture them as `vault.layout.<key> = path` entries. Otherwise return defaults:

```
{
  inbox: "+ Inbox",
  daily: "Calendar/Daily",
  weekly: "Calendar/Weekly",
  meetings: "Calendar/Meetings",
  oneOnOnes: "Calendar/1-1s",
  projects: "Efforts/Projects",
  areas: "Efforts/Areas",
  outcomes: "Efforts/Outcomes",
  people: "People",
  teams: "People/Teams",
  archive: "Archive",
  templates: "_Meta/Templates",
  adrs: "Atlas/Engineering"
}
```

### `## People`
Required for any skill that uses person-tier weighting (`/morning-start`, `/gmail-triage`, `/action-items-compress`, `/action-items-triage`).

Parse:
- `### Importance Tiers` — a markdown table with columns `Tier | Weight | Role | Members`. Members are `@`-prefixed. Build a map `personTiers[<name>] = { tier: N, weight: W }`. Strip `@` from names but preserve the wikilink form when emitting back.
- `### Direct Reports` — bullet list of `@Name` entries with optional descriptions. Return as `directReports: [{ name, description }]`.
- `### Key Stakeholders` — same shape as direct reports → `stakeholders: [{ name, description }]`.

If `### Importance Tiers` table is missing or malformed, return an error with: "Could not parse `## People` → `### Importance Tiers` table. Expected columns: Tier | Weight | Role | Members."

### `## Channels (Slack)`
Required for `/morning-start`, `/morning-brief`, `/blocker-scan`. Parse subsection headers (`### Daily Pulse`, `### Deploy / Alerts`, `### Leadership`, etc.) as channel groups. Each bullet is `#channel-name [— purpose]` with an optional `(ID: C0XXXXXX)` suffix.

Return:
```
channels: {
  "daily-pulse": [{ name: "team-platform", id: null, purpose: "main team channel" }, ...],
  "deploy-alerts": [{ name: "alerts-platform-deploys", id: "C0XXXXXX", purpose: "..." }],
  "leadership": [...]
}
```

Group keys are slugified subsection headers (lowercase, spaces and slashes → hyphens).

If the section is missing entirely, return `channels: {}` and a `warnings` entry: "No `## Channels` section in CLAUDE.md — Slack-using skills will skip the Slack scan."

### `## Projects / Initiatives`
Optional. Parse bullets as `[{ name, description, dashboardPath? }]`. Skills use these to cross-reference Slack/email mentions to active work.

### `## Gmail GTD Labels`
Required for `/gmail-triage` and the email portion of `/morning-start`. Parse bullets as `[{ label, purpose }]`. The `label` is the backticked string (e.g., `📥 GTD/1 - Next Actions/@Email`); `purpose` is the text after the `—`.

If missing, return `labels: []` and a warning: "No `## Gmail GTD Labels` section — `/gmail-triage` cannot classify."

### `## Notion`
Optional. Parse `[Page Title](url) — what's there` bullets as `[{ title, url, description }]`. Used for cross-reference lookups.

### `## Conventions`
Parse bulleted key-value pairs. Recognize at minimum:
- "Daily note path"
- "Weekly note path"
- "Person notes" path
- "Meeting ignore patterns" (comma-separated list)
- "Stakeholder audiences" (comma-separated list)
- "Status values" (comma-separated list)
- "Tags" (comma-separated list)

Return as `conventions: { dailyNotePath: "...", meetingIgnorePatterns: [...], ... }`.

## Output shape

```json
{
  "status": "ok" | "missing" | "error",
  "warnings": ["..."],
  "vault": { "layout": { ... } },
  "people": {
    "tiers": {
      "1": { "weight": 5, "role": "CEO", "members": ["Maya Patel"] },
      ...
    },
    "personTiers": { "Maya Patel": { "tier": 1, "weight": 5 }, ... },
    "directReports": [{ "name": "Riley Cohen", "description": "..." }],
    "stakeholders": [...]
  },
  "channels": { ... },
  "projects": [...],
  "labels": [...],
  "notion": [...],
  "conventions": { ... }
}
```

## Behavior on errors

- Missing `CLAUDE.md` → return `status: "missing"` with a single clear message; the calling skill will surface it to the user.
- Malformed required section → return `status: "error"` with the specific section name and what was expected.
- Missing optional section → return populated parts of the object plus a `warnings` entry.

Never invent data. If the user wrote `TODO` or left placeholders, treat them as empty.

## Contract for calling skills

Every workflow skill that loads `vault-config` should:

1. **On `status: "missing"`** — stop the skill immediately and surface this exact message to the user:
   > "No `CLAUDE.md` at vault root. Copy `CLAUDE.md.template` to `CLAUDE.md`, fill in your people / channels / projects / GTD labels, then re-run this skill. See `CLAUDE.md.example` for a filled-out reference."

2. **On `status: "error"`** — stop and report the specific section that failed validation, with the expected format. Do not try to proceed with partial config for required sections.

3. **On `status: "ok"` with `warnings`** — continue, but include a "Gaps" line in the output noting what's missing and which features will be skipped (e.g., "Skipped Slack scan — no `## Channels` section in CLAUDE.md").

This contract keeps the user-facing failure mode consistent across the suite.

## Caching

Within a single command invocation, parse once and reuse. Other skills calling you in the same session can reference the parsed object without re-reading `CLAUDE.md`.
