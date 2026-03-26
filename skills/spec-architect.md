---
name: spec-architect
description: Leads the transition from a vague idea to a solid specification. Combines Discovery, Grill-Me, AC-Generator, and the plan template into a single end-to-end workflow. Use when the user says "I want to build X" or "Start a new spec for Y."
---

Guide a new idea through grounding, challenge, specification, and handoff — in that order.

## Trigger

User says "I want to build X" or "Start a new spec for Y."

## Execution Protocol

### Phase 1: Grounding (Discovery)

Run the `discovery` skill on the current codebase before asking the user anything.

- Identify existing patterns, modules, or data structures affected by the new request.
- Goal: avoid asking questions the code can already answer.

### Phase 2: The Grilling (Grill-Me)

Initiate the `grill-me` protocol. Focus questions on:

- State Changes: "If we add this, how does the global state change?"
- Edge Cases: "What happens if the network fails at Step 2?"
- Principle Alignment: "This feature suggests a global store, but `steering/principles.md` forbids it. How shall we reconcile this?"

For every question asked, provide a recommended answer grounded in the `steering/` documents.

Continue until shared understanding is reached across all branches of the decision tree.

### Phase 3: Spec Drafting (AC-Generator)

Once shared understanding is reached, invoke the `acceptance-criteria-generator` skill.

- Draft `specs/requirements.md` and `specs/acceptance-criteria.md`.
- Ensure every branch explored during the grilling is represented as a testable criterion.

### Phase 4: Handoff (Plan Template)

Fill out `templates/plan/plan-template.md` based on the agreed spec.

- Present the plan to the user for sign-off.
- Once approved, move the resulting tasks into `inbox/tasks.md`.
