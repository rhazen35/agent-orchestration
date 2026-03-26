---
name: sentinel
description: A low-cost pre-checker that runs before the quality-reviewer. Catches cheap, obvious failures early to avoid burning expensive high-reasoning model calls on trivial mistakes.
---

Run fast, cheap checks on a completed task before escalating to the `quality-reviewer`. If any check fails, return the task to the worker immediately without invoking the inspector.

## Trigger

A task has been moved to `outbox/{namespace}/pending-review.md` and is awaiting quality review.

## Execution Protocol

Run all checks in order. Stop and reject on the first failure.

### Check 1: Syntax / Lint

- Does the changed code pass the project's linter or compiler without errors?
- If no linter is configured, check for obvious syntax errors manually.

### Check 2: Ghost Comments

- Are there any unfinished-work comments introduced by this task?
- Flag any: `TODO`, `FIXME`, `HACK`, `// I will finish this later`, or similar.
- If yes, reject — the task is incomplete.

### Check 3: Changelog Format

- If `CHANGELOG.md` was modified, is the new entry correctly formatted under `## [Unreleased]`?
- Does it use a valid category (Added / Changed / Fixed / Removed / Deprecated)?

### Check 4: Task-to-File Match

- Does the task summary match the actual files modified?
- If the implementation touched files not mentioned in the task, flag as scope creep.

## Outcomes

- Pass: All checks green → escalate to `quality-reviewer`.
- Fail: Any check red → move task back to `inbox/{namespace}/tasks.md` with a `[SENTINEL_FAIL]` tag specifying which check failed. Do not invoke `quality-reviewer`.

## Sentinel Dumb Loop

If the `sentinel` fails 3 consecutive times on the same task, do not reject again. Instead, apply a `[CONTINUE]` pause per `steering/governance.md` — this is a Dumb Loop and requires human intervention.

## Constraints

- Do not perform deep reasoning or spec comparison — that is the `quality-reviewer`'s job.
- Keep this skill fast and cheap. Binary pass/fail only.

## Project Overrides

This skill can be extended or replaced at the project level. Create `.agent/skills/sentinel.md` in your project root to override this default. See `steering/overrides.md` for the resolution order and extension pattern.
