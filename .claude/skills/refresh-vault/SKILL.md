---
name: refresh-vault
description: |
  Detects information drift (older notes contradicting newer ones) and
  DRY violations (same content duplicated across notes instead of linked)
  across the vault, then applies targeted corrections automatically.
  Mechanical fixes apply directly; editorial fixes apply with stricter
  verification and revert if not source-backed. Produces a changelog
  the user reviews after. Triggers: "refresh the vault", "audit consistency",
  "/refresh-vault".
model: claude-opus-4-7
effort: max
allowed-tools: Read Write Edit Grep Glob Bash
---

# /refresh-vault — Drift + DRY auditor

Detect information drift and DRY violations, apply targeted corrections automatically, write a post-execution report.

## Context

This vault uses the ACE framework with Obsidian-flavored markdown. Conventions are documented in `CLAUDE.md` — read it first.

Notes are interconnected via wikilinks. When one note is updated with new information (a metric, a project status, a reporting line), related notes may still contain the old information.

## Algorithm

Execute in order. Be thorough but efficient — read strategically, not exhaustively.

Apply the `vault-config` contract first (`missing` / `error` / `warnings`). If `CLAUDE.md` is missing, surface the standard setup message and stop — this skill is a no-op without conventions to anchor on.

### Step 1 — Build the file freshness map

Run a bash command to list every `.md` file (excluding `.claude/`, `.trash/`, `_Meta/Templates/`) sorted by modification time desc.

```
find . -name "*.md" -not -path "./.claude/*" -not -path "./.trash/*" -not -path "./_Meta/Templates/*" -printf "%T@ %p\n" | sort -rn
```

**If the find returns fewer than 8 files** (essentially empty vault), this skill has nothing to compare against. Report "Vault is too small for a meaningful consistency audit yet — come back once you have ~10+ notes" and exit. For vaults between 8 and 20 files, run normally but expect sparse output — drift detection improves as cross-references accumulate.

Group into three tiers:
- **Tier 1 (Source of Truth):** modified in the last 24 hours
- **Tier 2 (Recent):** modified in the last 7 days
- **Tier 3 (Stale candidates):** modified more than 7 days ago

### Step 2 — Read Tier 1 files; extract key facts

For each Tier 1 file, extract a structured list of **verifiable facts** that could contradict older notes:
- Metrics and numbers (adoption rates, user counts, revenue, NPS)
- Project statuses (phase, milestone, completion)
- Dates and deadlines (launches, rollout phases, OKR periods)
- People assignments (who owns what, reporting lines, team membership)
- Strategic positions (roadmap sequence, priorities, architectural decisions)
- Tool/technology choices

Store as your **truth set**.

### Step 3 — Identify cross-referenced notes

For each Tier 1 file, extract all wikilinks and backlinks. These linked notes most likely contain outdated information. Also check Tier 2 wikilinks to Tier 1.

### Step 4 — Audit linked notes against the truth set

Read each cross-referenced note (prioritize Tier 3, then Tier 2). Flag any **concrete discrepancy**:
- Number/metric differs
- Status outdated
- Date/deadline changed
- Person's role/team/responsibility changed
- Strategic direction contradicted
- Roadmap shifted phases or reprioritized
- Section references information that has been superseded

**Do NOT flag:**
- Differences in level of detail
- Stylistic or formatting differences
- Intentionally historical notes (meeting notes, daily notes, archive)
- Placeholder sections empty by design

### Step 5 — Check for orphaned or broken cross-references

- Wikilinks in `related:` frontmatter pointing to non-existent notes
- Notes that discuss a topic extensively but don't link to its canonical note
- `related:` frontmatter missing links that clearly should be there

### Step 6 — Check MOC completeness

Read all `Atlas/MOCs/*.md`. For each, verify:
- Every active project/area in `<vault.layout.projects>` is represented
- Every relevant `Atlas/` knowledge note is linked
- Descriptions match current state
- No dead links to moved or renamed notes

### Step 7 — DRY check

Scan for **substantive content duplication**:

1. **Paragraph-level duplication:** same explanation/context/analysis in 2+ notes (not just same fact, same prose). Replace secondary instance with link/embed.
2. **Identify the canonical source:**
   - Atlas/ > Efforts/ > Calendar/ > People/
   - Within same folder: most complete version
   - If equal: most recently modified
3. **List/table duplication:** same list of people, stakeholders, priorities maintained in multiple places. One note owns; others reference (`![[Note#Section]]` or link).
4. **Metric/stat duplication:** same number hardcoded in multiple notes. Metrics live in one canonical note; references elsewhere.

**NOT duplication:**
- Brief contextual summaries in daily/meeting notes that link to full source (good practice)
- Frontmatter fields that naturally repeat (`tags`, `related`)
- Task items appearing in originating note + queried via Dataview elsewhere
- Same person in multiple `attendees` lists across meetings

**How to fix:**
- Replace with link: `See [[Canonical Note#Section]]`
- Replace with embed: `![[Canonical Note#Section]]`
- Replace hardcoded metric with reference to canonical note
- Consolidate: if neither copy is canonical, merge and link

**Scope:** focus on Tier 1 and Tier 2 files. Tier 3 lower priority — they may already be stale and caught by Steps 2–4.

### Step 8 — Classify findings

Every finding is **Mechanical** or **Editorial**.

**Mechanical** — corrected value is unambiguously asserted in canonical Tier 1/2 source. No judgment, no synthesis, no prose composition. Examples:
- Frontmatter field updates where canonical source asserts the value verbatim
- Date corrections where the canonical source asserts the date
- Strikethroughs of departed people (where person note carries `status: departed`)
- Removing duplicate rows / dead links
- Replacing reference to archived note with `superseded-by` target

**Editorial** — correction requires composing prose, inferring values, restructuring, or judgment. Examples:
- Rewriting a paragraph to reflect current state
- Adding a "Status Update" or "Departure Transition" block
- Filling empty frontmatter field with a value not canonically asserted (e.g., picking an `owner:` no source explicitly names)
- Adding new frontmatter field type the vault hasn't used
- Synthesizing a list/roster that didn't previously exist
- Collapsing stale detail into summary + link

Tally both: `[mechanical]` and `[editorial]`.

### Step 9 — Execute fixes automatically

Apply both classes without asking. Edit notes preserving frontmatter (add fields, don't remove).

**Hard rules — both classes:**

1. **Do not fabricate.** If a value isn't canonically asserted in Tier 1/2, do not invent. Use placeholders that signal uncertainty: `TBD`, `unknown`, or omit. **Do not** infer specific dates from phrases like "end of week" or "~05/01" — write `~2026-05-01 (per [source])` so the approximation is visible.
2. **Do not infer reporting lines.** Reports-to fields are load-bearing. If the canonical org doesn't explicitly assert a manager, write `TBD`.
3. **Do not fill empty frontmatter fields** unless a canonical source explicitly names the value.
4. **Do not introduce new frontmatter field types** the vault hasn't used. Add a body section instead.
5. **Do not duplicate rosters/lists you found elsewhere.** Link to the canonical, don't re-list.
6. **Cite the source for every editorial change** — name Tier 1/2 file and section the new prose was synthesized from.

### Step 10 — Verify edits

After all edits, run a verification pass.

**Mechanical fixes:** re-grep each changed file to confirm the substitution applied.

**Editorial fixes (stricter):** re-read both edited file and cited canonical source. Confirm:
1. Every factual claim in the new prose is supported verbatim or by direct paraphrase
2. No values synthesized from "feels right" inference (dates, names, routing, ownership)
3. No new DRY violation introduced

Failed editorial verification → **revert it** and add to the report under "Reverted — needs human review."

### Step 11 — Post-execution report

Write to `<vault.layout.daily>/YYYY-Mon/refresh-vault-YYYY-MM-DD.md`:

- Total fixes applied, split by `[mechanical]` and `[editorial]`
- Each editorial fix: file, section, new prose (or summary), cited source
- Reverted fixes with the verification failure
- "Needs Human Decision" findings — items requiring judgment beyond the editorial bar (e.g., resolving contradiction between two equally-fresh sources, deciding which person owns an orphaned responsibility, deciding whether to delete vs. archive). Recommended approach but no edit applied.

The user reads after the run. Verification is the safety net.

## Important rules

- **Never modify** `<vault.layout.daily>`, `<vault.layout.oneOnOnes>`, `<vault.layout.meetings>` — historical records (read as truth-set inputs, don't edit).
- **Never modify** `<vault.layout.archive>/` — intentionally frozen.
- **Preserve frontmatter** — add or update; never remove existing fields. Don't introduce new field types.
- **Maintain wikilink format** — `[[Note Name]]`, `[[@Person]]`.
- **Ask before creating new notes** — propose in the report; don't create without approval.
- **Read CLAUDE.md first** — conventions may have changed.
- **Don't fabricate values** — see Step 9 hard rules.
- **Cite sources for every editorial change** and verify the source supports the claim before keeping the edit.
- **Beware of self-introduced DRY violations** — when fixing a stale list, prefer linking to canonical over re-listing.
