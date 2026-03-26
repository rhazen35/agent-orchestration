---
inclusion: always
---

# The Factory Pipeline

This document defines the end-to-end workflow for this project. Every phase must be explicitly triggered by the user — no phase runs automatically. This prevents unintended compute costs and keeps the human in control of each transition.

---

## Namespacing

Every specification lives in its own namespace — a dedicated folder used consistently across all phases of the pipeline. This keeps specs, plans, tasks, and records isolated from one another and prevents cross-contamination between features.

When a new spec is started, derive a short kebab-case name (e.g., `rate-limiter`, `user-auth`) and apply it as the namespace across all relevant directories:

| Directory | Namespaced path |
|---|---|
| `specification/` | `specification/{namespace}/requirements.md` |
| `specification/` | `specification/{namespace}/acceptance-criteria.md` |
| `plan/` | `plan/{namespace}/plan.md` |
| `inbox/` | `inbox/{namespace}/tasks.md` |
| `outbox/` | `outbox/{namespace}/pending-review.md` |
| `outbox/` | `outbox/{namespace}/completed.md` |
| `records/` | `records/{namespace}/spec-evolution.log` |
| `records/` | `records/{namespace}/ADR-XXX-*.md` |
| `archive/` | `archive/{namespace}/decision-log.md` |

All file references within skills and templates should resolve relative to the active namespace. When invoking any skill, the namespace must be established first.

---

## Governance

All agents operate under the hard limits defined in `steering/governance.md`. These rules are non-negotiable:

- Inbox hydration limit: max 3 active tasks per namespace at any time.
- Loop limit: a task rejected 3 times triggers a Hard Halt for human resolution.
- Token budget: if a task exceeds the reasoning budget, pause and ask the human to split it.
- Cooldown: after 5 consecutive completions, wait for human "Continue" before proceeding.
- Discovery caching: always check `cache/project-fingerprint.json` before re-scanning the codebase.
- JIT context: the `task-processor` reads only the task, the file it edits, and `steering/principles.md`.

---

## Overview

```
Design → Queue → Production → Sentinel → Quality Control → Archiving
```

Each phase has a designated skill, input, output, and a required user action to proceed.

---

## Phase 1: Design

Goal: Transform a vague idea into a signed-off specification and plan.

Skill: `spec-architect` (orchestrates `discovery`, `mcp-orchestrator`, `grill-me`, `acceptance-criteria-generator`)

Steps:
1. User says "I want to build X" or "Start a new spec for Y."
2. Run `discovery` on the codebase — identify affected patterns, modules, and data structures.
3. Run `mcp-orchestrator` — discover available MCP tools, match against task requirements, enrich spec with live data where applicable.
4. Run `grill-me` — challenge assumptions on state changes, edge cases, and principle alignment. Provide a recommended answer for every question.
5. Once shared understanding is reached, run `acceptance-criteria-generator` to produce:
   - `specification/{namespace}/requirements.md` — functional requirements
   - `specification/{namespace}/acceptance-criteria.md` — testable criteria using `templates/acceptance-criteria-template.md`
6. Fill out `templates/plan/plan-template.md` to produce `plan/{namespace}/plan.md`.
7. Log any significant architectural decisions as an ADR in `records/` using `templates/records/ADR-001-template.md`.
8. Append a change entry to `records/{namespace}/spec-evolution.log` per `steering/audit-requirements.md`.

Trigger: User explicitly invokes `spec-architect`.
Gate: User must sign off on `plan/plan.md` before proceeding to Phase 2.

---

## Phase 2: Queue

Goal: Break the approved plan into atomic tasks and load the inbox in small batches.

Steps:
1. Extract the task list from `plan/{namespace}/plan.md`.
2. Load the first batch of maximum 3 tasks into `inbox/{namespace}/tasks.md`.
3. Remaining tasks stay in the plan until the inbox has capacity (see `steering/governance.md` — Inbox Hydration Limit).

Trigger: User explicitly says "queue tasks" or "load inbox" after signing off on the plan.
Gate: User confirms `inbox/{namespace}/tasks.md` looks correct before proceeding to Phase 3.

---

## Phase 3: Production

Goal: Implement tasks one at a time with minimal context (JIT).

Skill: `task-processor` (may invoke `mcp-orchestrator` in lightweight mode)

Steps:
1. Pick the top-most task from `inbox/{namespace}/tasks.md`.
2. Load only: the task, the file(s) being edited, and `steering/principles.md`. Nothing else.
3. If the task requires external data, invoke `mcp-orchestrator` in lightweight mode — check `archive/mcp-data/{namespace}/` first, call a tool only if data is missing or stale.
4. Implement strictly — no unrelated refactoring.
5. Move the completed task entry to `outbox/{namespace}/pending-review.md`.
6. Repeat for the next task only when explicitly asked.

Trigger: User explicitly says "process next task" or invokes `task-processor`.
Gate: Each task must be moved to `outbox/{namespace}/pending-review.md` before the next begins.

---

## Phase 3.5: Sentinel Pre-Check

Goal: Catch cheap, obvious failures before invoking the expensive quality reviewer.

Skill: `sentinel`

Steps:
1. Run syntax/lint check on changed code.
2. Verify `CHANGELOG.md` entry format if modified.
3. Check for leftover `TODO`, `FIXME`, or `HACK` comments.
4. Check for scope creep — files touched outside the task's defined scope.
5. Pass: escalate to Phase 4.
6. Fail: return task to `inbox/{namespace}/tasks.md` with a `[SENTINEL REJECTION]` note. Do not invoke `quality-reviewer`.

Trigger: Runs as part of the "review" command, before `quality-reviewer`.
Gate: Task must pass Sentinel before `quality-reviewer` is invoked.

---

## Phase 4: Quality Control

Goal: Deep reasoning review — only runs on code that has passed the Sentinel.

Skill: `quality-reviewer` (preceded by `sentinel`)

Steps:
1. Pick the top-most task from `outbox/{namespace}/pending-review.md`.
2. Compare code changes against `specification/{namespace}/acceptance-criteria.md` and `steering/principles.md`.
3. Run `integrity-check` to detect ghost features, missing intent, or principle violations.
4. Pass: Move the task to `outbox/{namespace}/completed.md`.
5. Fail: Move the task back to `inbox/{namespace}/tasks.md` with a `[REJECTION NOTE]`. Increment `[LOOP COUNT]` on the task. If loop count reaches 3, apply `[HARD HALT]` per `steering/governance.md`.

Trigger: User explicitly says "review" or invokes `quality-reviewer`.
Gate: Task must be in `outbox/{namespace}/completed.md` before proceeding to Phase 5.

---

## Phase 5: Archiving

Goal: Record progress permanently, clean up workflow folders, and update the control tower.

Skill: `changelog-architect` + `monitor`

Steps:
1. Read the task summary and Git hash from `outbox/{namespace}/completed.md`.
2. Categorize the change (Added / Changed / Fixed / Removed / Deprecated).
3. Append an entry to `CHANGELOG.md` under `## [Unreleased]`.
4. Append the task entry and review notes to `archive/{namespace}/decision-log.md`.
5. Delete the task entry from `outbox/{namespace}/completed.md`.
6. Invoke `monitor` to update `dashboard.md` with the latest metrics and activity log entry.

Trigger: User explicitly says "archive" or invokes `changelog-architect`.

---

## Supporting Documents

These are used throughout the pipeline and are not tied to a single phase:

| Document | Role |
|---|---|
| `steering/principles.md` | Architectural constraints enforced in every phase |
| `steering/governance.md` | Hard limits: loop cap, token budget, cooldown, hydration limit, caching |
| `steering/audit-requirements.md` | Requires a `spec-evolution.log` entry and ADR for every requirement change |
| `records/{namespace}/spec-evolution.log` | Audit trail of all requirement changes |
| `records/{namespace}/ADR-XXX.md` | Architecture Decision Records for significant technical choices |
| `templates/records/ADR-001-template.md` | Template for new ADRs |
| `templates/acceptance-criteria-template.md` | Template for acceptance criteria documents |
| `templates/plan/plan-template.md` | Template for the plan document |
| `cache/project-fingerprint.json` | Fingerprint used to determine if a discovery re-scan is needed |
| `cache/discovery-report.md` | Cached output of the last `discovery` run and MCP capability map |
| `archive/{namespace}/mcp-data/` | Persisted MCP tool outputs — reused by Worker to avoid re-calling tools |
| `archive/{namespace}/decision-log.md` | Permanent record of completed tasks and review notes |
| `CHANGELOG.md` | Human-readable release history |
| `dashboard.md` | Live control tower — tracks token usage, cost, throughput, and circuit breaker state |

---

## Manual Trigger Summary

No phase runs automatically. Use these explicit commands:

| Command | Phase triggered |
|---|---|
| "I want to build X" / "Start a new spec for Y" | Phase 1: Design |
| "Queue tasks" / "Load inbox" | Phase 2: Queue |
| "Process next task" | Phase 3: Production |
| "Review" | Phase 3.5 + 4: Sentinel → Quality Control |
| "Archive" | Phase 5: Archiving |
