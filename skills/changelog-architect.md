---
name: changelog-architect
description: Provides a permanent, human-readable record of progress by appending completed tasks to CHANGELOG.md and cleaning up the workflow folders. Use when tasks are present in outbox/completed.md.
---

Record completed work in `CHANGELOG.md` and clean up `outbox/completed.md`.

## Trigger

Presence of tasks in `outbox/completed.md`.

## Execution Protocol

1. Read the task summary and associated Git hash from `outbox/completed.md`.
2. Categorize the change using one of the standard types below.
3. Append an entry to `CHANGELOG.md` under the `## [Unreleased]` section.
4. Append the task entry and any review notes to `archive/decision-log.md` for future reference.
5. Delete the task entry from `outbox/completed.md`.

## Change Categories

- `Added` — New functionality introduced.
- `Changed` — Modification to existing behavior.
- `Fixed` — Bug or defect resolved.
- `Removed` — Functionality intentionally removed.
- `Deprecated` — Functionality marked for future removal.

## Changelog Entry Format

```
### Added / Changed / Fixed / Removed / Deprecated
- [TASK-ID] Description of the change. (git: <hash>)
```

## Constraints

- Only append to the `## [Unreleased]` section — never modify versioned entries.
- Keep descriptions concise and human-readable.
- Always clean up `outbox/completed.md` after appending.
