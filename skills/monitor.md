---
name: monitor
description: Updates dashboard.md after every task cycle. Tracks token usage, cost, throughput, circuit breaker state, and generates hourly optimization suggestions. Invoked by the changelog-architect at the end of each archiving step.
---

Maintain `dashboard.md` as the live control tower for the pipeline. Keep it accurate so agents and humans can assess system health at a glance.

## Trigger

- After every task cycle (invoked by `changelog-architect` at the end of Phase 5).
- Hourly, to generate optimization suggestions.
- Whenever a circuit breaker condition is detected.

## Execution Protocol

### Step 1: Pre-Flight Check

Before updating, read the current `dashboard.md`.

- If status is 🔴 CIRCUIT BREAKER TRIPPED, do not proceed — surface a `[HARD HALT]` to the human.
- If status is 🟡 THROTTLED, note the throttle in the update but continue.

### Step 2: Update Resource Consumption

Update the metrics table in Section 1:

- Total Tokens: cumulative tokens used this session.
- Estimated Cost: derived from token count.
- Active Tasks: count of tasks currently in `inbox/{namespace}/tasks.md`.
- Uptime: elapsed time since session start.

If Estimated Cost increased by more than $0.50 in a single task cycle, flip System Status to 🟡 THROTTLED.

### Step 3: Update Pipeline Throughput

Update Section 2:

- Increment Tasks Completed.
- Recalculate Average Rejections per Task.
- Recalculate Sentinel Block Rate (sentinel rejections / total tasks attempted).

### Step 4: Update Circuit Breakers

Update Section 3:

- List any tasks with `[LOOP COUNT: 2]` as warnings (1 attempt remaining).
- If any task has hit the Rule of Three (4th rejection), flip System Status to 🔴 CIRCUIT BREAKER TRIPPED.
- Update Budget Guard percentage.

### Step 5: Append Activity Log Entry

Prepend a new line to Section 5 in this format:

```
[HH:MM] {Agent}: {Action} — {Task ID} ({outcome}).
```

Keep the log to the 20 most recent entries.

### Step 6: Hourly Optimization Suggestion

If an hour has elapsed since the last suggestion, analyze the current metrics and append a suggestion to Section 4.

Examples:
- Sentinel Block Rate > 20%: "Worker is failing basic checks frequently. Consider clarifying `steering/principles.md` or adding a linting config."
- Average Rejections > 1.0: "Tasks are being rejected more than once on average. Consider breaking tasks into smaller units during Phase 2."
- Single task used > 30k tokens: "Task complexity is high. Review batch sizing in Phase 2."

## Constraints

- Never delete historical activity log entries beyond the 20-entry rolling window.
- Never reset metrics without explicit human instruction.
- Status may only be reset to 🟢 OPERATIONAL by a human.
