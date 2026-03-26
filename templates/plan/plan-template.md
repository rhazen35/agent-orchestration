# Plan: [Title]

## 1. Intent & Context

- Target Spec: Link to the specific file in `specs/` being addressed.
- Problem Statement: A brief, high-level summary of what we are solving.
- Discovery Insights: Summarize what was found in the codebase that affects this plan (e.g., "Existing UserStore already has a `find()` method we can leverage").

## 2. Proposed Architectural Changes

- New Components: List any new modules or interfaces to be created.
- Modified Components: List existing files that will be changed.
- Data Flow: Describe how data will move through the system for this feature (Input → Logic → Output).

## 3. Adherence to Steering

- Principle Alignment: How does this plan follow `steering/principles.md`? (e.g., "Ensuring logic remains pure by injecting the database dependency").
- Boundary Protection: Explicitly state how you are preventing "leakage" between layers.

## 4. Step-by-Step Execution

Detailed, atomic steps. Each step should be small enough to verify individually.

- [ ] Task 1: (e.g., Define the interface in `domain/interfaces`)
- [ ] Task 2: (e.g., Implement the core logic in `domain/services`)
- [ ] Task 3: (e.g., Hook into the delivery layer / API)

## 5. Verification Plan

- Success Criteria: How will we know this works? (Matches the acceptance criteria in `specs/`).
- Integrity Check: What specific checks will be run after implementation to ensure no drift occurred?
