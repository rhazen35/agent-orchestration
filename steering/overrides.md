---
inclusion: always
---

# Skill & Steering Overrides

This package provides defaults for every skill, steering rule, and template. Any consuming project can extend or override them without modifying the package files directly.

## Resolution Order

When loading any skill, steering file, or template, the agent must check in this order:

1. `.agent/{directory}/{filename}` — project-level override (takes precedence)
2. `{directory}/{filename}` — package default (fallback)

Examples:
- `.agent/skills/quality-reviewer.md` overrides `skills/quality-reviewer.md`
- `.agent/steering/principles.md` overrides `steering/principles.md`
- `.agent/templates/plan/plan-template.md` overrides `templates/plan/plan-template.md`

---

## What Can Be Overridden

### Skills — all overridable

Every skill can be overridden. Different projects have different stacks, conventions, and tooling.

| Skill | Common reason to override |
|---|---|
| `acceptance-criteria-generator` | Domain-specific criteria formats or rules |
| `integrity-check` | Project-specific architectural patterns |
| `quality-reviewer` | Compliance checklists or team review rules |
| `sentinel` | Project-specific lint tools or syntax checks |
| `discovery` | Custom codebase mapping for unusual structures |
| `task-processor` | Project-specific implementation constraints |
| `spec-architect` | Custom spec workflow or domain-specific grilling |
| `grill-me` | Domain-specific assumption challenges |
| `changelog-architect` | Custom changelog formats |
| `monitor` | Custom dashboard metrics or reporting |

### Steering — selectively overridable

| File | Override? | Notes |
|---|---|---|
| `principles.md` | ✅ Yes | Every project has its own architecture |
| `audit-requirements.md` | ✅ Yes | Adjust audit format or log location |
| `governance.md` | ⚠️ With caution | You may adjust token budgets and batch sizes, but keep the structure intact |
| `overrides.md` | ❌ No | This file defines the override system itself |
| `workflow.md` | ❌ No | The pipeline contract — changing it breaks the factory pattern |

### Templates — all overridable

| Template | Common reason to override |
|---|---|
| `templates/plan/plan-template.md` | Custom plan structure |
| `templates/acceptance-criteria-template.md` | Custom criteria format |
| `templates/records/ADR-001-template.md` | Custom ADR format |

---

## How to Override

Create a `.agent/` directory in your project root mirroring the structure of the package:

```
your-project/
  .agent/
    skills/
      quality-reviewer.md
      sentinel.md
    steering/
      principles.md
      governance.md
    templates/
      plan/
        plan-template.md
```

---

## How to Extend (Partial Override)

To add to the default behavior rather than replace it entirely, start your override file with:

```markdown
# [Skill Name] — Project Extension

> Extends the base `skills/{skill-name}.md`. All base behavior applies unless explicitly overridden below.

## Project-Specific Additions

...your additions here...
```

---

## What Should NOT Be Overridden

- `steering/workflow.md` — changing the pipeline order breaks the factory pattern
- `steering/overrides.md` — this file defines the override system itself
