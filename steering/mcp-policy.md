---
inclusion: always
---

# MCP Policy

This file defines the capability requirements and tool selection logic for this factory. It does not list specific MCP servers — those are configured by the host environment. Instead, it defines what the agent expects to find and how it behaves when tools are present or absent.

---

## The Handshake

When the agent starts, it does not read a config file to discover what it can do. It performs a handshake with the host:

1. Agent calls the native `list_tools` command.
2. Host returns a list of available tool schemas based on its own configuration.
3. Agent matches the returned schemas against this policy to determine which tools are authorized for the current task.

If a required capability is missing from the `list_tools` output, the agent must fall back to manual user input — never guess or hallucinate a tool response.

---

## Capability Requirements

The following capabilities are expected from the host environment. Tools fulfilling these capabilities will be authorized for use. Tools outside these categories require explicit human approval before use.

| Capability | Purpose | Fallback if missing |
|---|---|---|
| Search | Verify documentation, library versions, external references | Ask the human to provide the information manually |
| Filesystem | Read/write `specification/`, `inbox/`, `archive/` | Use local file tools already available in the agent environment |
| Runtime | Execute tests and linting (primary tool for the `sentinel`) | Mark sentinel checks as manual and surface to human |
| Database | Inspect live schemas for spec validation | Use schema definitions in `specification/` if available |
| Version Control | Read issues, PRs, or commit history linked to requirements | Use local `records/` and `archive/` as reference |

---

## Tool Selection Logic

When multiple tools are returned by `list_tools` that fulfill the same capability, the agent must prioritize in this order:

1. Local filesystem tools — lowest latency, zero external cost.
2. Specialized local servers (e.g., local Postgres, SQLite, local search index).
3. Remote API servers (e.g., GitHub, Brave Search, Slack).

Never call a remote API tool when a local tool can fulfill the same need.

---

## Authorization Rules

A tool is authorized for a task if:

- It matches one of the capability categories above.
- Its use is justified by a specific gap in `specification/`, `archive/`, or `cache/`.
- It passes the Local-First check from `steering/mcp-etiquette.md`.

A tool is not authorized if:

- It performs write, delete, or push actions without a sentinel pre-check or human `[CONTINUE]`.
- It falls outside the capability categories above without explicit human approval.
- The information it would provide already exists locally.

---

## Missing Capabilities

If a required capability is absent from `list_tools`, the agent must:

1. Note the missing capability in `dashboard.md` under Active Circuit Breakers.
2. Fall back to the defined fallback for that capability.
3. Never block the pipeline — proceed with what is available.
