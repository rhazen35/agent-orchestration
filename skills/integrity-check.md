---
name: integrity-check
description: Systematically audit the codebase against specs and principles to detect drift, ghost features, and architectural violations. Use before finalizing a plan, after completing tasks, or when asked to "verify the state of the project".
---

Audit the current codebase and project state against `specs/` and `steering/principles.md`. Prevent feature creep and ensure architectural purity by identifying where code has diverged from intent.

## Triggers

- Pre-Plan: Before a `plan.md` is finalized.
- Post-Task: After a set of atomic tasks in `tasks.md` are marked complete.
- Audit Request: When the user explicitly asks to "verify the state of the project."

## Execution Protocol

### Step 1: Specification Alignment

Compare the current codebase against the active files in `specs/`.

- Ghost Features: Flag any code, logic, or utility that exists but was never defined in a spec.
- Missing Intent: Flag any requirement in the spec that does not have a corresponding implementation.

### Step 2: Principle Verification

Audit the latest changes against `steering/principles.md`.

- Boundary Check: Ensure no low-level details (persistence, UI, external APIs) are leaking into the domain logic.
- Dependency Check: Verify that high-level modules are not importing low-level concrete implementations.
- Side-Effect Check: Ensure new functions are pure and state transitions are explicit.

### Step 3: Output Report

Generate a concise Integrity Audit Report with these sections:

- Alignment: `[Pass/Fail]` — Does the code match the specs?
- Principle Adherence: `[Pass/Fail]` — Does the code follow the steering rules?
- Detected Drift: List any ghost features or accidental complexities added.
- Violations: List specific breaches (e.g., "Leaky abstraction in UserModule").
- Correction Plan: What specific refactors are needed to restore integrity?

## Success Criteria

- The code reflects exactly the intent of `specs/`.
- The codebase maintains the architecture defined in `steering/`.
- No magic or hidden global states were introduced.
