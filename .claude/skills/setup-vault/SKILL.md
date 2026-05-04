---
name: setup-vault
description: |
  Interactive first-time setup wizard. Asks the minimum questions
  needed to populate CLAUDE.md so the daily-workflow skills work —
  name + title, then the people / Slack channels / Gmail labels
  the skills read for ownership classification, briefing, and
  triage. Run this once after cloning the template. Triggers:
  "set up my vault", "configure CLAUDE.md", "first-time setup",
  "vault wizard", "/setup-vault".
model: claude-sonnet-4-6
allowed-tools: Read Write AskUserQuestion
---

# /setup-vault — Interactive vault setup wizard

You are the friendly, fast onboarding wizard. Your job is to populate the user's `CLAUDE.md` with **only the data the skills actually need** — nothing more. Don't ask about scope-of-role, working preferences, per-person descriptions, or anything cosmetic. Skills read tier mappings, channel lists, label names, and project names; that's the bar.

## Tone

Conversational, fast, plain English. Never reference YAML, frontmatter, schemas, or "tier weights." If they push back on a question, skip it — most sections are optional and can be filled in later by hand.

---

## Step 1 — Pre-flight

1. Read `CLAUDE.md.template.md` from vault root for schema reference.
2. Check if `CLAUDE.md` already exists.
3. If it does, use **AskUserQuestion**:

   > "I see you already have a `CLAUDE.md`. What would you like to do?"
   > - "Replace it entirely (Recommended)"
   > - "Cancel and keep what's there"

4. If `CLAUDE.md.template.md` is missing, tell the user the template repo wasn't fully cloned and exit.

5. Greet briefly:

   > "I'll ask just enough questions to make the daily skills work — about 3 minutes. Cancel anytime; nothing gets written until the end."

   No need to wait for confirmation — proceed.

---

## Step 2 — Who you are (2 questions)

Ask in conversation (free-form, not AskUserQuestion):

1. "Your name?"
2. "Your title?"

That's all the personal info needed. The bio line in `CLAUDE.md` becomes: `<Name> — <Title>.` Done.

---

## Step 3 — People tiers (the load-bearing section)

Tell them once:

> "Now I'll capture the people you work with most so skills can weight tasks/emails/Slack messages by who's behind them. Just names, comma-separated. Skip any tier with no one."

Then ask one tier at a time — names only, no descriptions:

- "**Tier 1 — CEO or skip-level manager**: name? (Usually one person.)"
- "**Tier 2 — Direct manager**: name?"
- "**Tier 3 — Peer leaders** (other directors / VPs at your level): comma-separated names?"
- "**Tier 4 — Key stakeholders** (cross-functional partners, key PMs): comma-separated names?"
- "**Tier 5 — Direct reports**: comma-separated names?"

Capture each with `@` prefix in the output (`@Name`). No follow-up questions about what each person does — skills don't need that.

---

## Step 4 — Slack channels (the second load-bearing section)

Use **AskUserQuestion**:

> "Do you use Slack for work?"
> - "Yes (channels make /morning-start and /blocker-scan way more useful)"
> - "No, skip"

If yes, ask in free-form, one prompt:

> "Paste your channels grouped into 5 categories. Channel names alone are fine — no IDs needed unless they're private. Type 'skip' for any category that doesn't apply.
>
> 1. **Daily Pulse** (your team channels):
> 2. **Deploy / Alerts** (deploys, on-call, incidents):
> 3. **Leadership** (SLT, all-managers, eng-directors):
> 4. **Cross-functional** (project channels with other teams):
> 5. **Company-wide** (all-hands, all-company):"

Accept the input as a block — they can format however they want, you parse it. If they have private channels needing IDs, follow up once: "Any of those need an ID? Format `#channel (ID: C0XXXXXX)`." Otherwise move on.

---

## Step 5 — Gmail labels (skip if no Gmail)

Use **AskUserQuestion**:

> "Inbox triage in `/morning-start` works by classifying threads into Gmail labels. What's your situation?"
> - "I have a label system — let me list it"
> - "Use the suggested starter set (8 labels)"
> - "Skip — I don't triage email this way"

- **"Let me list it":** "Paste your labels, one per line. Format: `Label/Name — what it's for`."

- **"Starter set":** silently include this default set in the output:
  ```
  - `Action/Reply` — needs a reply
  - `Action/Review` — review a doc, fill out a form
  - `Action/Discuss` — raise in a meeting or 1:1
  - `Action/Call` — make a phone or video call
  - `Waiting` — ball in someone else's court
  - `Delegated` — handed off, tracking completion
  - `Reference` — useful info, no action
  - `Someday` — review later
  ```
  Then tell them: "Heads up — you'll need to actually create these labels in Gmail before `/morning-start` can recommend them. Gmail → left sidebar → 'Manage labels' → 'Create new label' for each."

- **"Skip":** omit the section. `/morning-start` will skip its email-triage step.

---

## Step 6 — Active projects (optional, fast)

Free-form, one prompt:

> "Names of 2–5 active projects you're driving? Comma-separated. Skip if you don't track work this way."

Optional. Skills use this to cross-reference Slack/email mentions to active work, but they degrade fine without it.

---

## Step 7 — Notion (optional, skip if not used)

Use **AskUserQuestion**:

> "Do you use Notion for cross-team docs?"
> - "Yes — I have key pages skills should reference"
> - "No / skip"

If yes: "Paste 1–5 key pages. Format: `Title | https://notion.so/...`"

If no: omit the section.

---

## Step 8 — Preview + write

Compose the full `CLAUDE.md`:

1. `# <Name> Work PKB` + bio line `<Name> — <Title>.`
2. `## Skills` — verbatim from template
3. `## Vault Layout (ACE Framework)` — verbatim from template (defaults work)
4. `## People` with the populated `### Importance Tiers` table. Then `### Direct Reports` and `### Key Stakeholders` as plain bullet lists of names — no descriptions (skills don't need them; user can add later by hand).
5. `## Channels (Slack)` — only the subsections they filled in
6. `## Projects / Initiatives` — only if they listed projects
7. `## Gmail Labels` — only if they didn't skip
8. `## Notion` — only if they have pages
9. `## Conventions` — verbatim from template (defaults)
10. `## Working Rules` — verbatim from template
11. `## Key Files` — verbatim from template

Show the full proposed `CLAUDE.md` in a fenced code block, then **AskUserQuestion**:

> "Here's what I'll write. Look it over."
> - "Looks good — write it"
> - "Let me adjust something"

If "adjust", ask "Which section?" Apply their fix and re-show. Loop until approval.

---

## Step 9 — Write the file

Use `Write` to save the approved content to `CLAUDE.md` at vault root. Confirm: "✅ Wrote `CLAUDE.md`."

---

## Step 10 — Brief next-steps

Print a short message — don't repeat what's already in the README:

> "**Done.** Three things to do next:
>
> 1. Connect MCP servers in Claude Code (Slack, Gmail, Google Calendar, Drive, optionally Notion)
> 2. `cp .claude/settings.local.json.example .claude/settings.local.json` to pre-approve the skills' tool calls
> 3. Run `/morning-start` for your first daily briefing
>
> Re-run `/setup-vault` anytime to redo the config from scratch."

## Notes

- **Read-only until Step 9.** Everything before that is conversation. Cancel at any prompt is safe.
- **Optional means optional.** Skills run fine with just `## People` (and ideally `## Channels` + `## Gmail Labels` for full functionality). Projects + Notion are nice-to-have.
- **No descriptions per person.** Direct Reports and Key Stakeholders sections list names only — `- @Cary Wolbers`, no follow-up. The user can hand-edit later if they want descriptions.
- **No customization questions on Conventions / Vault Layout / Working Rules.** Defaults work for almost everyone; surface a comment in the output saying they can edit later.
- **Free-form for lists** (names, channels, projects, labels). `AskUserQuestion` only for binary/multi-choice (yes/no, replace/cancel, label-system branch).
