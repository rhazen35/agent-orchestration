---
name: task-processor
description: Executes raw tasks from the planning phase strictly without deviating from the spec or second-guessing decisions. Use when tasks are present in inbox/tasks.md.
---

Execute tasks from `inbox/tasks.md` one at a time, strictly following the spec and principles. Do not interpret, expand, or refactor beyond what the task specifies.

## Trigger

Presence of tasks in `inbox/tasks.md`.

## Execution Protocol

1. Pick the top-most task from `inbox/tasks.md`.
2. Implement the change strictly following `steering/principles.md`.
3. Once the code change is made, move the task entry from `inbox/tasks.md` to `outbox/pending-review.md`.

## Constraints

- Do not refactor unrelated code.
- Do exactly what the task specifies — no more, no less.
- Do not second-guess or deviate from the spec.
