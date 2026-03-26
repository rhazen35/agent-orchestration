---
inclusion: always
---

# Principles of Development

This document defines the core philosophies and architectural constraints for all implementation tasks. These principles supersede any specific library "shortcuts" or temporary fixes.

## 1. Separation of Concerns

- Logic over Framework: Core business logic (domain rules) must remain decoupled from the delivery mechanism (Web, CLI, API) and the persistence layer (Database, File System).
- Pure Functions: Favor pure functions that take inputs and return outputs without side effects.
- Single Responsibility: Each module, class, or function must have one, and only one, reason to change.

## 2. Interface-First Design

- Contractual Boundaries: Define how components talk to each other (Inputs/Outputs) before writing the internal implementation.
- Depend on Abstractions: High-level modules should not depend on low-level modules. Both should depend on abstractions (interfaces).
- Explicit over Implicit: Avoid "magic" behavior or global states. Dependencies must be passed explicitly (Dependency Injection).

## 3. The "Spec-to-Code" Integrity

- No Ghost Features: If a feature or logic branch is not defined in `specs/`, it must not exist in the code.
- Traceability: Every major function or module should be traceable back to a specific requirement in a specification file.
- Refactor, Don't Patch: If a new requirement breaks an existing pattern, refactor the pattern to accommodate the change rather than adding an `if` statement.

## 4. Error Handling and Resilience

- Fail Fast: Validate inputs at the boundary. If data is malformed, reject it immediately with a clear reason.
- Error as Value: Prefer returning errors as data structures rather than throwing unhandled exceptions that crash the execution flow.
- Total Functions: Aim for functions that handle all possible input states, including "empty," "null," or "error" cases.

## 5. Readability and Maintainability

- Self-Documenting Code: Use descriptive naming for variables and functions. A function name should tell you what it does; the code should tell you how.
- Small Surface Area: Keep the public API of any module as small as possible. Hide internal implementation details (encapsulation).
- DRY vs. AHA: Avoid premature abstraction. It is better to have duplicate code than the wrong abstraction. Only abstract when a clear pattern emerges (Avoid Hasty Abstractions).
