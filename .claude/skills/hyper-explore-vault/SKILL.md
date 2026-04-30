---
name: hyper-explore-vault
description: |
  Performs an exhaustive, multi-agent analysis of the vault. Reads every
  file across the ACE framework regions, builds a wikilink graph, and
  produces a synthesis report covering structural health, narrative arc,
  hidden threads, contradictions, gaps, and a prioritized action queue.
  Triggers: "deep audit my vault", "what does my vault tell you", "/hyper-explore-vault".
model: claude-opus-4-7
effort: max
allowed-tools: Read Glob Grep Bash Task TaskOutput
---

# /hyper-explore-vault — Deep multi-agent vault audit

Exhaustive, maximally parallel deep analysis. Read every file. Discover structure, meaning, and hidden threads. Thoroughness over speed.

## Phase 1 — Full inventory

Get a complete picture before reading anything.

1. **Glob all file types** in parallel:
   - `**/*.md` — all markdown files
   - `**/*.canvas` — all canvas files
   - `**/*.base` — all Bases files
2. **Count and catalog:**
   - Total files by type
   - Directory tree (every folder, file count each)
   - Top-level ACE regions: `Atlas`, `Calendar`, `Efforts`, `People`, `Archive`, `_Meta`, `+ Inbox`, root files
3. **If glob results are truncated**, fall back to `bash find` for complete listing. 100% coverage required — no skipped files.

Output: structured inventory with file counts per directory. Keep as your working reference.

## Phase 2 — Parallel deep read (6 agents)

Launch **6 background agents in parallel** using the Task tool (`subagent_type: "general-purpose"`, `run_in_background: true`). Each agent reads every file in its assigned region.

**Each agent prompt must include the full file list it should read** (from Phase 1). Don't rely on agents to discover files themselves.

**Fallback if the Task tool isn't available:** if the harness disallows background agents (e.g., older Claude Code, restricted permissions), fall back to running each region's analysis sequentially in this skill's main context. Skip Phase 3 (collection) and merge findings inline as you go.

**Fresh-vault note:** on a sparsely populated vault (under ~50 files), the multi-agent parallelism is overkill. Detect this in Phase 1 (`total_files < 50`) and run sequentially regardless — the synthesis is still useful but cheaper.

### Agent 1: Atlas & Knowledge Base
Files: everything under `Atlas/`. Analyze:
- Topics covered? Knowledge graph structure?
- Which MOCs exist and what do they link to? Gaps in MOC coverage?
- Architectural decisions documented? Technical stack?
- Competitive/market intelligence?
- Depth and completeness of each knowledge area (shallow stub vs. rich)

### Agent 2: Calendar & Temporal
Files: everything under `Calendar/`. Analyze:
- Date range covered? Cadence?
- Narrative arc — what story do daily notes tell day-by-day?
- Most-frequent meeting attendees? Recurring topics?
- Decisions made? Deferred?
- Energy/emotional trajectory?
- Empty directories that should have content?

### Agent 3: People & Network
Files: everything under `People/`. Analyze:
- Person note count? Stub vs. rich context ratio?
- Org hierarchy from the notes: reporting lines, teams
- Most cross-referenced people across the vault
- People mentioned elsewhere but missing person notes (stub gaps)
- Naming inconsistencies (nicknames vs. formal)
- Team notes well-connected vs. orphaned

### Agent 4: Efforts, Projects & Areas
Files: everything under `Efforts/`. Analyze:
- Active projects + status
- Areas of responsibility defined?
- OKRs/outcomes — populated or empty?
- Project ↔ broader strategic context
- Project ↔ people/teams references
- Implied priority stack from detail and cross-references

### Agent 5: Root Files & Structure
Files: all root-level `.md` files (`Home`, `Action Items`, etc.) + `_Meta/Templates/`, `Archive/`, `+ Inbox/`. Analyze:
- Vault's "operating system" — how do root files orchestrate daily workflow?
- Templates and the patterns they establish
- Inbox contents (unprocessed)
- What's archived and why
- Action Items ↔ daily notes/meetings relationship
- Slash commands or automation referenced

### Agent 6: Wikilink Graph & Cross-References
Files: ALL files (complete list from Phase 1). Analyze:
- Connectivity map: which notes link to which
- Hub notes (highest outgoing + incoming)
- Orphan notes (no inbound links)
- Dead links (wikilinks to non-existent files)
- Clusters — heavily cross-referenced groups with few links to other clusters
- Naming inconsistencies in wikilinks (`[[Person]]` vs `[[@Person]]`)
- Link density per region

## Phase 3 — Wait & collect

Wait for all 6 agents. Collect findings via TaskOutput.

## Phase 4 — Synthesis report

Combine into a single comprehensive report:

### I. Structural Health
- Vault score (1–10) with justification
- File count and distribution across ACE regions
- Dead links (full list with locations)
- Orphan notes (full list)
- Missing notes (entities referenced but without their own note)
- Empty or stub notes that need content

### II. The Meaning
- What is this vault actually about? What story does it tell?
- Narrative arc across the timeline
- Attention mapping — where is the vault owner spending time based on note density and cross-references?
- Implicit OKRs — what goals are the notes working toward, even if not formally stated?
- What does the vault say the owner is worried about?

### III. Hidden Threads
**Implicit connections** — patterns spanning multiple notes/initiatives but not explicitly linked. The "aha" insights that only emerge from reading everything.

For each thread:
- Memorable name
- Connection with specific evidence from vault notes
- Why it matters
- Connected notes

Look for:
- Multiple notes independently pointing to the same conclusion
- Tensions between what different notes say about the same topic
- Dependencies no single note captures
- People or topics appearing across unrelated contexts
- Conclusions the vault implies but never states

### IV. Strategic Contradictions
Tensions or contradictions. Different notes implying different priorities; stated goals conflicting with observed patterns.

### V. What's Missing
- Person notes that should exist but don't
- Topics referenced but never given their own note
- Connections that should be wikilinked but aren't
- Entire categories of knowledge absent given the vault owner's role
- Process gaps (e.g., empty `Calendar/Weekly/`)

### VI. Priority Queue
Highest-leverage actions the vault owner should take. Rank by impact.

## Phase 5 — Offer artifacts

Offer to create (with user approval — knowledge base writes require approval):

1. **Hidden Threads MOC** (`Atlas/MOCs/Hidden Threads MOC.md`) — discovered threads wikilinked, with a Mermaid dependency graph
2. **Stub person notes** for missing people from Agent 3
3. **Dead link fix instructions** (`+ Inbox/Dead Link Fixes.md`) — step-by-step with paths, line numbers, before/after
4. **Vault Health canvas** — visual node graph: clusters, orphans, hubs, hidden threads, problem areas

Wait for approval before creating.
