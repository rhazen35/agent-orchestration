---
inclusion: always
---

# MCP Etiquette

Rules that govern how the agent behaves when MCP tools are available. These rules exist to prevent tool distraction and ensure the local filesystem remains the authoritative source of truth.

---

## The Local-First Rule

Before using any tool discovered via `list_tools`, the agent must check whether the information already exists locally:

1. Check `specification/{namespace}/` — does the spec already answer this?
2. Check `archive/mcp-data/{namespace}/` — has this tool already been called and cached?
3. Check `cache/discovery-report.md` — is this covered by the last discovery run?

Only if all three are empty or stale should the agent proceed with a tool call.

> Tools augment the spec-driven process. They do not replace the local source of truth.

---

## Tool Use is Purposeful, Not Exploratory

The agent must not call tools out of curiosity or to "see what's there." Every tool call must be justified by a specific gap in the local spec or a concrete task requirement.

- Allowed: "The spec references library X. I need to verify the current stable version. `brave_search` is available and the local files don't have this."
- Not allowed: "I have `postgres` available. Let me explore the database to see if there's anything interesting."

---

## Tools Do Not Overwrite Specs

MCP tool output must never directly overwrite or replace content in `specification/`. It may only:

- Inform a human-reviewed update to the spec.
- Be persisted to `archive/mcp-data/` for reference.
- Be summarized and injected as context for the current task.

The human or the `spec-architect` skill makes the final call on spec changes.

---

## Silence is Acceptable

If no MCP tools are available or relevant, the agent must proceed with local files only. The absence of tools is not an error — it is the default state. The pipeline works without any MCP servers running.
