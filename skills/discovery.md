---
name: discovery
description: Systematic protocol for understanding an unfamiliar or evolving codebase. Builds a mental map of architecture, data flow, and dependencies before proposing any plan. Use when first introduced to a project, after a major context shift, or when spec references don't clearly map to known implementations.
---

Build a thorough understanding of the codebase before proposing any changes. Map the architecture, data flow, and dependencies systematically.

## Triggers

- Initialization: When the agent is first introduced to the project.
- Context Shift: When a new high-level requirement is added to `specs/`.
- Ambiguity: When the agent finds a reference in a spec that doesn't clearly map to a known implementation.

## Execution Protocol

### Step 1: Structural Mapping

- Tree Analysis: Generate a directory map. Identify where "Logic," "Data," and "Delivery" (UI/API) are separated.
- Pattern Recognition: Identify which structural pattern is currently implemented (e.g., Layered, Hexagonal).
- Entry Points: Identify the primary bootstrap files where the system starts.

### Step 2: Dependency Graphing

- Internal Dependencies: Map how modules talk to each other. Which module is the "Core" (no dependencies) and which are "Periphery"?
- External Boundaries: Identify all external points of contact (Databases, Third-party APIs, File Systems).
- Contract Analysis: Locate where interfaces or abstract definitions live vs. where their concrete implementations are.

### Step 3: Data Lifecycle Trace

- Input to Output: Trace a single piece of data from the entry point to the storage/output layer.
- State Management: Locate where "Truth" is stored. Is it centralized, or distributed across modules?

### Step 4: Discovery Report

Produce a Project Map with these sections:

- Current Architecture: Summarize the structural style found.
- Core Domain: List the primary entities and logic centers found.
- IO Boundaries: List all external systems the project interacts with.
- Deviations: Note any areas where the current code violates `steering/principles.md`.

## Success Criteria

- The agent can explain how a specific user journey in `specs/` is currently fulfilled by the code.
- The agent identifies "No-Go Zones" where refactoring would be high-risk.
- The agent can list all side effects caused by a primary function.
