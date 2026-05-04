---
date: 2025-01-01
tags:
  - person
  - sample
role: Senior Engineer
team: Platform
reports-to: "@Direct Manager"
email: example@company.com
related:
---
# @Example Person

**Role:** Senior Engineer
**Team:** Platform
**Reports to:** @Direct Manager

> [!note] Sample file
> Delete this and create real person notes using `_Meta/Templates/Person.md`. The intentionally-omitted `last-contact` / `contact-frequency` fields below are how your real notes drive the "Stale Relationships" widget in your daily note.

## Context

How you work with this person. What matters to them.

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

---

> [!tip] Stale Relationships widget
> When you create your real person notes from the Person template, add these frontmatter fields to participate in the daily note's stale-relationships tracking:
>
> ```yaml
> last-contact: 2026-04-15      # date you last spoke
> contact-frequency: 14          # cadence in days; widget flags when overdue
> ```
>
> Omit them on this sample note so it doesn't generate noise on day 1.
