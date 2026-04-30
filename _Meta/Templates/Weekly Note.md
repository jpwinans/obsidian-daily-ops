---
date: {{date:YYYY-MM-DD}}
tags: weekly
week: {{date:YYYY-[W]ww}}
status: active
---
# Week of {{date:MMMM D, YYYY}}

## Accomplishments

-

## In Progress

-

## Blockers

-

## Incomplete Tasks

```tasks
not done
due before {{date+7d:YYYY-MM-DD}}
group by due
```

## Decisions Made This Week

```dataview
LIST
FROM #decision
WHERE date >= date("{{date:YYYY-MM-DD}}") - dur(7 days)
SORT date DESC
```

## Next Week Focus

1.
2.
3.

## Reflection

**What went well:**
**What to improve:**
