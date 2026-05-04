---
date: {{date:YYYY-MM-DD}}
tags:
  - daily
status: active
---
# {{date:dddd, MMMM D, YYYY}}

## Morning Briefing

> [!tip] Top 3 Items Needing Attention
> 1. **—**
> 2. **—**
> 3. **—**

*Run `/morning-brief` or `/morning-start` to auto-populate this section.*

### Overdue Tasks

```dataview
TASK
WHERE due < date(today) AND !completed
SORT due ASC
```

### Due Today

```dataview
TASK
WHERE due = date(today) AND !completed
```

### Stale Relationships

```dataviewjs
const pages = dv.pages('"People"')
  .where(p => p["last-contact"] && p["contact-frequency"])
  .where(p => {
    const last = dv.luxon.DateTime.fromISO(p["last-contact"].toString());
    const freq = p["contact-frequency"];
    const daysSince = dv.luxon.DateTime.now().diff(last, 'days').days;
    return daysSince > freq;
  })
  .sort(p => p["last-contact"], 'asc');

if (pages.length > 0) {
  dv.table(
    ["Person", "Last Contact", "Days Overdue"],
    pages.map(p => {
      const last = dv.luxon.DateTime.fromISO(p["last-contact"].toString());
      const days = Math.floor(dv.luxon.DateTime.now().diff(last, 'days').days);
      const overdue = days - p["contact-frequency"];
      return [p.file.link, p["last-contact"], overdue];
    })
  );
} else {
  dv.paragraph("All relationships current.");
}
```

### Inbox

```dataview
LIST FROM "+ Inbox"
SORT file.ctime ASC
```

---

## Focus

> What is the ONE thing that matters most today?

-

## Tasks

- [ ]

## Meetings

### 00:00 - Meeting 1

## Capture

## End of Day

- [ ] Process Inbox
- [ ] Update task dates
- [ ] Log wins

**Energy:** /5
**Key decision made:**
**Carried forward:**
