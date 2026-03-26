# rhazen35/agent-orchestration

A tool-agnostic agent orchestration setup for spec-driven development. Provides a complete "Factory Pipeline" workflow with skills, steering rules, templates, and governance guardrails that keep AI agents productive without burning your token budget.

This package is designed to be dropped into any project — or required via Composer — to give your AI agent a structured, auditable way to go from idea to shipped code.

---

## Installation

This package contains no runtime code — only markdown files consumed by your AI agent. It works with any language or stack.

### PHP (Composer)

```bash
composer require rhazen35/agent-orchestration
```

The package lands in `vendor/rhazen35/agent-orchestration/`. Copy the directories you need into your project root:

```bash
cp -r vendor/rhazen35/agent-orchestration/skills ./skills
cp -r vendor/rhazen35/agent-orchestration/steering ./steering
cp -r vendor/rhazen35/agent-orchestration/templates ./templates
cp -r vendor/rhazen35/agent-orchestration/records ./records
```

Or reference the files directly from `vendor/` if your agent tooling supports path configuration.

### Node / npm

```bash
npm install rhazen35-agent-orchestration
```

Files land in `node_modules/rhazen35-agent-orchestration/`. Copy what you need:

```bash
cp -r node_modules/rhazen35-agent-orchestration/skills ./skills
cp -r node_modules/rhazen35-agent-orchestration/steering ./steering
cp -r node_modules/rhazen35-agent-orchestration/templates ./templates
cp -r node_modules/rhazen35-agent-orchestration/records ./records
```

Or point your agent tooling directly at `node_modules/rhazen35-agent-orchestration/`.

### Any Other Language (Git Submodule)

For Go, Rust, Python, Node, or any other stack, use a Git submodule. This keeps the orchestration setup versioned and updatable without a language-specific package manager.

```bash
git submodule add https://github.com/rhazen35/agent-orchestration .agent
```

This places the full setup in `.agent/` inside your project. Point your agent tooling at `.agent/steering/`, `.agent/skills/`, etc.

To update to the latest version later:

```bash
git submodule update --remote .agent
```

To clone a project that already uses this as a submodule:

```bash
git clone --recurse-submodules <your-repo-url>
```

### No Package Manager (Quick Start)

If you just want to get started without submodules:

```bash
git clone https://github.com/rhazen35/agent-orchestration .agent
```

Then add `.agent/` to your `.gitignore` if you don't want to track it, or commit it directly if you prefer a snapshot.

---

## What's Included

```
skills/          — Agent skills (invokable protocols)
steering/        — Always-on rules loaded into every agent context
templates/       — Reusable document templates
records/         — ADR template and spec evolution log
inbox/           — Tasks waiting to be processed
outbox/          — Tasks pending review or completed
archive/         — Permanent decision log
cache/           — Discovery cache (gitignored at runtime)
dashboard.md     — Live control tower for pipeline health
CHANGELOG.md     — Human-readable release history
```

---

## The Factory Pipeline

The core of this package is a five-phase pipeline. Every phase is manually triggered — nothing runs automatically. This keeps you in control and prevents runaway token costs.

```
Design → Queue → Production → Sentinel → Quality Control → Archiving
```

### Phase 1: Design

Trigger: say "I want to build X" or "Start a new spec for Y."

The `spec-architect` skill runs three sub-skills in sequence:

1. `discovery` — maps the codebase, identifies affected modules. Checks `cache/project-fingerprint.json` first to avoid redundant scans.
2. `grill-me` — challenges your assumptions on state changes, edge cases, and principle alignment. Provides a recommended answer for every question.
3. `acceptance-criteria-generator` — once shared understanding is reached, drafts `specification/{namespace}/requirements.md` and `specification/{namespace}/acceptance-criteria.md`.

Output: a signed-off `plan/{namespace}/plan.md` using `templates/plan/plan-template.md`.

Gate: you must sign off on the plan before anything moves to the inbox.

---

### Phase 2: Queue

Trigger: "queue tasks" or "load inbox."

Tasks from the plan are loaded into `inbox/{namespace}/tasks.md` in batches of maximum 3 (the Hydration Limit). The next batch only loads after a slot opens in the inbox.

Gate: confirm the inbox looks correct before production starts.

---

### Phase 3: Production

Trigger: "process next task."

The `task-processor` skill picks the top task and implements it using JIT context — it only reads the task, the file it's editing, and `steering/principles.md`. Nothing else. This keeps the context window small and each call cheap.

Output: task moved to `outbox/{namespace}/pending-review.md`.

---

### Phase 3.5: Sentinel Pre-Check

Runs automatically as part of the "review" command, before the expensive quality reviewer.

The `sentinel` skill checks for cheap failures:
- Syntax errors or lint failures
- Ghost comments (`TODO`, `FIXME`, unfinished notes)
- Changelog format errors
- Scope creep (files touched outside the task's scope)

If any check fails, the task goes back to `inbox/` with a `[SENTINEL_FAIL]` tag. The quality reviewer is never called.

---

### Phase 4: Quality Control

Trigger: "review."

The `quality-reviewer` skill performs deep reasoning — comparing the implementation against `specification/{namespace}/acceptance-criteria.md` and `steering/principles.md`. It also runs `integrity-check` to detect ghost features or architectural drift.

- Pass → task moves to `outbox/{namespace}/completed.md`.
- Fail → task returns to `inbox/` with a `[REJECTION NOTE]` and an incremented `[LOOP COUNT]`.

If a task is rejected 4 times, the pipeline locks and requires human intervention.

---

### Phase 5: Archiving

Trigger: "archive."

The `changelog-architect` skill:
1. Appends an entry to `CHANGELOG.md` under `## [Unreleased]`.
2. Appends the task and review notes to `archive/{namespace}/decision-log.md` for future debugging.
3. Cleans up `outbox/{namespace}/completed.md`.
4. Invokes `monitor` to update `dashboard.md`.

---

## Namespacing

Every specification lives in its own namespace folder. When starting a new spec, pick a short kebab-case name (e.g. `rate-limiter`, `user-auth`) and it applies across all directories:

| Directory | Path |
|---|---|
| Specification | `specification/{namespace}/` |
| Plan | `plan/{namespace}/` |
| Inbox | `inbox/{namespace}/tasks.md` |
| Outbox | `outbox/{namespace}/` |
| Records | `records/{namespace}/` |
| Archive | `archive/{namespace}/` |

---

## Governance

`steering/governance.md` defines hard limits that no agent can override:

- Inbox hydration limit: max 3 active tasks at once.
- Rule of Three: a task rejected 4 times locks the pipeline.
- Token budget: 50,000 tokens per run — Soft Halt if exceeded.
- Cooldown: after 5 consecutive completions, wait for human "Continue."
- Discovery caching: never re-scan if the fingerprint is current.
- JIT context: the worker reads only what it needs.

---

## Steering Files

These are loaded into every agent context automatically:

| File | Purpose |
|---|---|
| `steering/principles.md` | Core architectural constraints |
| `steering/governance.md` | Hard resource and loop limits |
| `steering/audit-requirements.md` | Requirement changes must be logged and ADR'd |
| `steering/workflow.md` | The full pipeline reference |

---

## Skills Reference

| Skill | When to use |
|---|---|
| `spec-architect` | Starting a new spec end-to-end |
| `discovery` | Mapping an unfamiliar codebase |
| `grill-me` | Stress-testing a plan or design |
| `acceptance-criteria-generator` | Turning requirements into testable criteria |
| `task-processor` | Implementing a task from the inbox |
| `sentinel` | Pre-checking a task before quality review |
| `quality-reviewer` | Deep review against spec and principles |
| `integrity-check` | Auditing the codebase for drift |
| `changelog-architect` | Archiving completed tasks |
| `monitor` | Updating the dashboard after each cycle |

---

## Dashboard

`dashboard.md` is the control tower. Check it before starting any work:

- 🟢 OPERATIONAL — all clear.
- 🟡 THROTTLED — wait for human `[CONTINUE]`.
- 🔴 CIRCUIT BREAKER TRIPPED — do not start work. Human reset required.

---

## Manual Trigger Summary

| Command | Phase |
|---|---|
| "I want to build X" | Phase 1: Design |
| "Queue tasks" | Phase 2: Queue |
| "Process next task" | Phase 3: Production |
| "Review" | Phase 3.5 + 4: Sentinel → Quality Control |
| "Archive" | Phase 5: Archiving |
