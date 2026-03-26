---
name: task-processor
description: Executes raw tasks from the planning phase strictly without deviating from the spec or second-guessing decisions. Use when tasks are present in inbox/tasks.md.
---

Execute tasks from `inbox/{namespace}/tasks.md` one at a time, strictly following the spec and principles. Do not interpret, expand, or refactor beyond what the task specifies.

## Trigger

Presence of tasks in `inbox/{namespace}/tasks.md`.

## Execution Protocol

1. Pick the top-most task from `inbox/{namespace}/tasks.md`.
2. Implement the change strictly following `steering/principles.md`.
3. If the task requires external data (live schema, API response, library version, etc.), invoke `mcp-orchestrator` in lightweight mode. Always check `archive/mcp-data/{namespace}/` for cached results before making any tool call.
4. Once the code change is made, move the task entry from `inbox/{namespace}/tasks.md` to `outbox/{namespace}/pending-review.md`.

## Constraints

- Do not refactor unrelated code.
- Do exactly what the task specifies — no more, no less.
- Do not second-guess or deviate from the spec.

## Project Overrides

This skill can be extended or replaced at the project level. Create `.agent/skills/task-processor.md` in your project root to override this default. See `steering/overrides.md` for the resolution order and extension pattern.
