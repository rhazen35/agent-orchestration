---
name: acceptance-criteria-generator
description: Transforms high-level functional requirements from specs/ into granular, binary (Pass/Fail) acceptance criteria. Use when a new requirement is created or updated, or as a mandatory step before drafting a plan.
---

Transform requirements from `specs/` into unambiguous, verifiable acceptance criteria. Use `templates/acceptance-criteria-template.md` as the output structure.

## Triggers

- Initialization: When a new requirement file is created in `specs/`.
- Modification: When an existing requirement is updated.
- Pre-Planning: As a mandatory step before `plan.md` can be drafted.

## Execution Protocol

### Step 1: Requirement Extraction

- Read the target file in `specs/`.
- Identify all "Must," "Should," and "Constraint" statements.
- Map out the "Happy Path" (the ideal user journey).

### Step 2: Scenario Mapping (Given / When / Then)

For every requirement identified, generate at least:

- One Positive Scenario: (e.g., "Given valid input, the system produces X.")
- Two Negative Scenarios: (e.g., "Given invalid input A, system returns Error Y"; "Given a system timeout, system returns Retry Z.")
- One Edge Case: (e.g., "Given a maximum length string, the system processes it without truncation.")

### Step 3: Quality Gate Integration

- Cross-reference `steering/principles.md`.
- Add a "Structural Integrity" section to the criteria (e.g., "Logic must be independent of external libraries," "Interfaces must be documented").

### Step 4: Output

- Append or update the specific feature section in `specs/acceptance-criteria.md`.
- Use the checklist format from `templates/acceptance-criteria-template.md`.

## Success Criteria

- Every requirement in the spec has at least one corresponding checklist item.
- All criteria are observable and verifiable — no vague terms like "fast" or "user-friendly."
- The output provides a complete "Definition of Done" for the implementation.
