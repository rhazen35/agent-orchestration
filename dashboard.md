# Agent Factory: Control Tower

System Status: 🟢 OPERATIONAL

> Updated by `monitor` skill after every task cycle. If status is 🔴, all agents must refuse to start new work until a human resets this file.

---

## 1. Resource Consumption (Current Session)

| Metric | Value | Limit | Status |
|---|---|---|---|
| Total Tokens | 0 | 500,000 | ✅ Healthy |
| Estimated Cost | $0.00 | $5.00 | ✅ Healthy |
| Active Tasks | 0 | 3 | ✅ Within Batch |
| Uptime | 0m | N/A | — |

---

## 2. Pipeline Throughput

- Tasks Completed: 0
- Average Rejections per Task: 0.0 (Target: < 1.0)
- Sentinel Block Rate: 0% (Target: < 20%)

---

## 3. Active Circuit Breakers

- Loop Protection: No tasks in warning state.
- Budget Guard: 0% of session quota used.

---

## 4. Optimization Suggestions

_No suggestions yet. Generated hourly based on Sentinel Block Rate and rejection patterns._

---

## 5. Recent Activity Log

_No activity yet._

---

## Pre-Flight Rules (Read Before Starting Any Task)

- If status is 🔴 CIRCUIT BREAKER TRIPPED — do not start work. Wait for human reset.
- If status is 🟡 THROTTLED — wait for human `[CONTINUE]` before processing the next task.
- If Estimated Cost increases by more than $0.50 in a single task — flip status to 🟡 automatically.
