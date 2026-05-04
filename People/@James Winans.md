---
date: 2025-01-01
tags:
  - person
  - peer
  - sample
role: Director of AI Product Engineering
team: AI Product
reports-to: "@Sam Chen"
email: james@example.com
last-contact: 2026-04-30
contact-frequency: 14
related:
  - "[[Observability Rollout]]"
---
# @James Winans

**Role:** Director of AI Product Engineering
**Team:** AI Product
**Reports to:** 

> [!note] Sample peer note
> Demonstrates a peer-leader person note (Tier 3) in the example "Acme Robotics" org. Delete or adapt to your own org.

## Context

How I work with this person. What matters to them.

- Peer Director — partners on shared infrastructure where AI Product and Platform overlap (eval tooling, tracing, gateway services).
- Strong preference for joint design over parallel stacks. Tends to surface duplicated work early.
- Schedules cross-team retros; values quarterly OKR alignment before drafts go up the chain.

## Notes

- Add running observations here as you work together.

## Meeting History

```dataview
TABLE date, file.link as "Note"
FROM "Calendar/1-1s" OR "Calendar/Meetings"
WHERE contains(attendees, this.file.name) OR contains(person, this.file.name)
SORT date DESC
LIMIT 10
```
