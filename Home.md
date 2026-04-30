---
tags:
  - home
status: active
---
# Home

Your vault entry point. Fill this in over time as your work takes shape.

## This Week

- See `Calendar/Weekly/` for the current weekly review
- Run `/rollup:weekly` to generate a fresh one

## Today

- Run `/morning:start` first thing in the morning
- Run `/prep:day` after that to prep upcoming meetings
- Run `/rollup:daily` at end of day

## Maps of Content

- [[Atlas/MOCs/Engineering MOC|Engineering]]
- [[Atlas/MOCs/Leadership MOC|Leadership]]
- [[Atlas/MOCs/Product MOC|Product]]
- (add more as your knowledge base grows)

## Active Work

```dataview
TABLE status, deadline
FROM "Efforts/Projects"
WHERE status = "active"
SORT deadline ASC
```

## Recent Daily Notes

```dataview
LIST
FROM "Calendar/Daily"
SORT file.ctime DESC
LIMIT 7
```

## Open Tasks Across the Vault

```tasks
not done
sort by due
limit 20
```

## Inbox

```dataview
LIST FROM "+ Inbox"
SORT file.ctime ASC
```

---

## Setup Checklist

- [ ] Filled in `CLAUDE.md` from `CLAUDE.md.template`
- [ ] Connected Slack MCP server
- [ ] Connected Gmail MCP server
- [ ] Connected Google Calendar MCP server
- [ ] (Optional) Connected Notion MCP server
- [ ] Created `People/@<your name>.md`
- [ ] Tried `/morning:start`
