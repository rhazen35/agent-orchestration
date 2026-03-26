---
inclusion: always
---

# Governance

Hard limits that apply to every agent in every phase. These rules are non-negotiable and override any skill or plan instruction.

---

## 1. Resource Guardrails

### Batch Size (Inbox Hydration Limit)

The inbox for any namespace must never contain more than 3 active tasks at once.

- The `spec-architect` must load tasks in batches of maximum 3.
- New tasks from `plan/{namespace}/plan.md` may only be hydrated into the inbox as slots become available (after a task reaches `outbox/completed.md`).
- Effect: A flawed plan fails fast and cheaply, not after 20 expensive API calls.

### Context Stripping (JIT Worker)

The `task-processor` is only permitted to read:

- The specific task entry from `inbox/{namespace}/tasks.md`.
- The specific file(s) it is editing.
- `steering/principles.md`.

It must not load the full `specification/`, `outbox/`, or any other namespace's files.

### Token Budget (Soft Halt)

Each agent run has a maximum budget of 50,000 tokens.

- If a task is projected to exceed this limit, the agent must Soft Halt: stop, surface the task to the human, and request it be split into two smaller tasks.
- Do not attempt partial implementations that leave the codebase in a broken state.

---

## 2. Loop & Error Controls (Circuit Breakers)

### The Rule of Three

A single task may only be bounced between `inbox/{namespace}/tasks.md` and `outbox/{namespace}/pending-review.md` a maximum of 3 times.

- Each rejection must increment a `[LOOP COUNT: N]` marker on the task entry.
- On the 4th rejection, the entire pipeline locks.
- A human must intervene to either rewrite the task or update `steering/principles.md` to resolve the ambiguity before the pipeline can continue.

### Validation Tiering

No high-reasoning model (`quality-reviewer`) may run until the low-cost `sentinel` has verified the code is syntactically correct and passes basic checks.

- If the `sentinel` fails 3 consecutive times on the same task, the pipeline must pause and wait for a manual `[CONTINUE]` — this indicates a Dumb Loop that human judgment must resolve.

---

## 3. Caching & State Management

### Discovery Cache

Before running the `discovery` skill, the agent must check `cache/project-fingerprint.json`.

- If the fingerprint is current (no files modified since last scan), use `cache/discovery-report.md` instead of re-scanning.
- If stale or missing, run `discovery` and update both cache files.
- This eliminates redundant full-codebase reads, which typically account for 80% of input token costs on large projects.

### Stateless Execution

Agents must not rely on chat memory for project context.

- Every piece of necessary information must be present in the filesystem: `specification/`, `steering/`, or `archive/`.
- If context is missing from the filesystem, surface the gap to the human rather than inferring from memory.

---

## 4. Human-in-the-Loop (HITL) Triggers

The pipeline must pause and wait for a manual `[CONTINUE]` command if any of the following occur:

- `CHANGELOG.md` is about to be updated with a "Breaking Change" category.
- The `sentinel` fails 3 consecutive times on the same task (Dumb Loop detected).
- The `spec-architect` proposes a change that conflicts with a protected file in `steering/`.
- A task hits the Rule of Three (4th rejection) — pipeline locks until human resolves.
- A Soft Halt is triggered by the token budget limit.
- 5 consecutive automated task completions occur without human interaction (Cooldown).

---

## 5. Dashboard & Pre-Flight

Before starting any new task, every agent must read `dashboard.md`.

- If System Status is 🔴 CIRCUIT BREAKER TRIPPED — refuse to start work. Wait for human reset.
- If System Status is 🟡 THROTTLED — wait for human `[CONTINUE]` before processing the next task.
- If Estimated Cost increases by more than $0.50 in a single task cycle — the `monitor` skill must flip status to 🟡 automatically.

The `monitor` skill is responsible for keeping `dashboard.md` current after every task cycle.
