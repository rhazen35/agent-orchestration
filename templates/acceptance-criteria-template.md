# Acceptance Criteria: [Feature Name]

- Linked Spec: (e.g., `specs/user-auth.md`)
- Status: `[Pending | In-Progress | Verified]`

## 1. Functional Scenarios

These describe the expected behavior of the system logic.

### Scenario 1: [Positive Path Name]

- Given: Initial state (e.g., "The system is empty")
- When: The action (e.g., "A valid data object is submitted")
- Then: The result (e.g., "The data is persisted and a unique ID is returned")

### Scenario 2: [Negative / Error Path Name]

- Given: (e.g., "The system already contains an ID with value 'X'")
- When: (e.g., "A submission with ID 'X' is attempted")
- Then: (e.g., "The system rejects the change with a 'Conflict' error code")

## 2. Boundary & Edge Cases

- [ ] Empty Inputs: How does the system handle null, empty strings, or empty lists?
- [ ] Large Payloads: Does the system handle data sizes at the upper limit defined in `specs/`?
- [ ] Concurrency: (If applicable) What happens if two identical requests arrive simultaneously?

## 3. Structural Integrity

- [ ] Zero Leakage: No implementation details from the delivery or persistence layers have bled into the core domain logic.
- [ ] Contract Adherence: The public interface matches the definition in `plan.md`.
- [ ] No Side Effects: The operation does not modify any state outside of its defined scope.
