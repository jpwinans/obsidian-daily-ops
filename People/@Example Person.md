---
date: 2025-01-01
tags:
  - person
role: Senior Engineer
team: Platform
reports-to: "@Direct Manager"
email: example@company.com
last-contact: 2025-01-01
contact-frequency: 14
related:
---
# @Example Person

**Role:** Senior Engineer
**Team:** Platform
**Reports to:** @Direct Manager

## Context

Sample person note. Delete this file and create your own using the Person template.

The `last-contact` and `contact-frequency` (in days) frontmatter fields drive the "Stale Relationships" Dataview query in your daily note.

## Notes

- Sample note about how to work with this person.

## Meeting History

```dataview
TABLE date, file.link as "Note"
FROM "Calendar/1-1s" OR "Calendar/Meetings"
WHERE contains(attendees, this.file.name) OR contains(person, this.file.name)
SORT date DESC
LIMIT 10
```
