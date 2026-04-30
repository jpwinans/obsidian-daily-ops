---
date: 2025-01-01
tags:
  - moc
  - leadership
status: active
---
# Leadership MOC

Map of Content for leadership and people-related notes.

## Org Chart

[[Atlas/Leadership/Org Chart]]

## People

```dataview
TABLE role, team, "reports-to" as "Reports to"
FROM "People"
WHERE !contains(file.path, "Teams/") AND !contains(file.path, "@Example")
SORT team, file.name
```

## Teams

```dataview
LIST FROM "People/Teams"
SORT file.name ASC
```

## Recent 1:1 Notes

```dataview
LIST
FROM "Calendar/1-1s"
SORT file.ctime DESC
LIMIT 10
```

## Related MOCs

- [[Atlas/MOCs/Engineering MOC|Engineering]]
- [[Atlas/MOCs/Product MOC|Product]]
