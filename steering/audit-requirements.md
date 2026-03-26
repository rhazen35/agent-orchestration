---
inclusion: always
---

# Requirement Change Auditing

Any time a requirement is added, modified, removed, or superseded, you MUST append an entry to `records/spec-evolution.log`.

Use this format:

```
[DATE] [CHANGE TYPE] [Reference] - Description
  Reason: Why the change was made
  Impact: What was affected
```

Valid change types: ADDED, MODIFIED, REMOVED, SUPERSEDED.

Do not skip this step. The audit trail exists to preserve the reasoning behind decisions for future contributors and agents.

## Architecture Decision Records

For any significant technical choice accompanying a requirement change, you MUST also create an Architecture Decision Record in `records/`. Use `templates/records/ADR-001-template.md` as the template and name the file `ADR-XXX-short-title.md`, incrementing the number from the last existing ADR.

An ADR is required when the change involves:
- Selecting a technology, library, or approach over alternatives
- Introducing a new architectural pattern or constraint
- Removing or superseding a previously accepted decision

Each ADR must include: status, date, context, decision, consequences, and alternatives considered.
