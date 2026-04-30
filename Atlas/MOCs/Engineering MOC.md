---
date: 2025-01-01
tags:
  - moc
  - engineering
status: active
---
# Engineering MOC

Map of Content for engineering knowledge in this vault. Update as new notes get created.

## Architecture & Decisions

```dataview
LIST
FROM "Atlas/Engineering"
WHERE contains(tags, "adr") OR contains(tags, "decision")
SORT date DESC
```

## Active Projects

```dataview
TABLE status, deadline
FROM "Efforts/Projects"
WHERE status = "active"
SORT deadline ASC
```

## Areas of Responsibility

```dataview
LIST FROM "Efforts/Areas"
SORT file.name ASC
```

## Recent Engineering Notes

```dataview
LIST
FROM "Atlas/Engineering"
SORT file.mtime DESC
LIMIT 10
```

## Related MOCs

- [[Atlas/MOCs/Leadership MOC|Leadership]]
- [[Atlas/MOCs/Product MOC|Product]]
