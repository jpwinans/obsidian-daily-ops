---
date: {{date:YYYY-MM-DD}}
tags:
  - person
role:
team:
reports-to:
email:
last-contact:
contact-frequency:
related:
---
# @{{title}}

**Role:**
**Team:**
**Reports to:**

## Context

How I work with this person. What matters to them.

## Notes

## Meeting History

```dataview
TABLE date, file.link as "Note"
FROM "Calendar/1-1s" OR "Calendar/Meetings"
WHERE contains(attendees, this.file.name) OR contains(person, this.file.name)
SORT date DESC
LIMIT 10
```
