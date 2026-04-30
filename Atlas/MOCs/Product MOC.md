---
date: 2025-01-01
tags:
  - moc
  - product
status: active
---
# Product MOC

Map of Content for product knowledge — competitive intelligence, market research, product strategy.

## Active Product Initiatives

```dataview
TABLE status, owner
FROM "Efforts/Projects"
WHERE contains(tags, "product") AND status = "active"
SORT file.name ASC
```

## Product Knowledge Notes

```dataview
LIST
FROM "Atlas/Product"
SORT file.mtime DESC
LIMIT 20
```

## Related MOCs

- [[Atlas/MOCs/Engineering MOC|Engineering]]
- [[Atlas/MOCs/Leadership MOC|Leadership]]
