---
name: decision
description: |
  Creates a properly numbered, well-structured ADR (Architecture Decision
  Record) from the current conversation context. Asks clarifying questions
  if context is insufficient. Triggers: "create an ADR", "log this decision",
  "/decision <title>".
model: claude-sonnet-4-6
argument-hint: "[Decision Title]"
allowed-tools: Read Write Glob Grep
---

# /decision — Architecture Decision Record (ADR)

Create a numbered ADR from the conversation context.

## Input

`$ARGUMENTS` = title of the decision (e.g., "Use LiteLLM Proxy for Model Aliasing").

## Instructions

### Step 1 — Read context

1. Load `vault-config` for `vault.layout` (specifically `vault.layout.adrs`, default `Atlas/Engineering`).
2. Load `obsidian-markdown` for syntax.
3. Read the ADR template at `<vault.layout.templates>/ADR.md`.

### Step 2 — Determine next ADR number

Scan `<vault.layout.adrs>/` for files matching `ADR-NNN*.md`. Find the highest number and increment by 1. If none exist, start at `ADR-001`.

### Step 3 — Extract decision context

Review the conversation for:
- **Context:** what prompted this decision? what problem are we solving?
- **The decision itself:** what was decided?
- **Alternatives considered:** what other options were evaluated?
- **Consequences:** what follows from this decision (positive, negative, risks)?
- **Deciders:** who was involved?
- **Related notes:** what projects, people, or knowledge notes connect?

### Step 4 — Ask clarifying questions

If the conversation doesn't provide enough, ask the user for:
- Alternatives considered and why they were rejected
- Key consequences (positive and negative)
- Deciders
- Status: `proposed`, `accepted`, `deprecated`

Do NOT proceed until you have sufficient context.

### Step 5 — Create the ADR

Write to `<vault.layout.adrs>/ADR-NNN Title.md`:

```yaml
---
date: YYYY-MM-DD
tags:
  - adr
  - decision
status: accepted
deciders:
  - "[[@Person Name]]"
related:
  - "[[Project or Note]]"
---
```

```markdown
# ADR-NNN: Decision Title

## Status

Accepted

## Context

[What is the issue motivating this decision? Include links to related notes.]

## Decision

[What was decided. Be specific and actionable.]

## Alternatives Considered

### Alternative 1: [Name]
[Description and why it was rejected]

### Alternative 2: [Name]
[Description and why it was rejected]

## Consequences

### Positive
- [What becomes easier or better]

### Negative
- [What becomes harder or worse]

### Risks
- [What could go wrong]
```

### Step 6 — Add backlinks

- Link all deciders with `[[@Name]]`
- Link all related projects with `[[Project Name]]`
- Link related existing ADRs if any
- Check if `<vault.layout.adrs>/Decision Log.md` exists — if so, note that it will auto-populate via Dataview

### Step 7 — Confirm

Report:
- File path created
- ADR number assigned
- Summary of the decision
- Any backlinks that reference notes not yet created (suggest creating them)
