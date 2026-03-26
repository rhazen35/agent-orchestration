---
inclusion: always
---

# MCP Governance

Rules of engagement for any MCP tool the agent discovers in the current environment. These rules apply regardless of which MCP servers are running — they govern how the agent interacts with any external tool capability.

---

## 1. Tool Discovery

Before starting any complex task, the agent must call `list_tools` to map the current environmental capabilities.

- Match available tools against the current task requirements.
- If a tool is available that can fulfill a spec requirement more accurately than local reasoning, prefer the tool.
- If no relevant tools are available, proceed with local files only — never block on missing MCP servers.
- Persist the tool map to `cache/discovery-report.md` alongside the codebase discovery report.

---

## 2. Context Budget

MCP tool outputs must be summarized or truncated if they exceed 2,000 tokens.

- Never inject raw, verbose tool output directly into the agent context.
- Summarize findings and persist the full output to `archive/mcp-data/{namespace}/{tool-name}-{date}.md`.
- The Worker reads from `archive/mcp-data/` rather than re-calling the tool.

---

## 3. Safety & State — Write/Delete/Push Controls

Any tool that performs a write, delete, push, or destructive action requires one of the following before execution:

- A `sentinel` pre-check confirming the action is within the task scope.
- Explicit human confirmation via a `[CONTINUE]` prompt.

Read-only tools (fetch, search, list, get) may be called freely within the token budget.

---

## 4. Reliability — Failure Fallback

If a tool call fails twice in a row:

- Do not retry a third time.
- Fall back to local reasoning using existing files in `specification/`, `steering/`, and `archive/`.
- Mark the task as `[BLOCKED]` in `dashboard.md` with a note explaining which tool failed and why.
- Surface the block to the human before proceeding.

---

## 5. Hallucination & Malformed Output Filter

Before any MCP tool output is used to update `specification/`, `steering/`, or any spec file, the `sentinel` must review it for:

- Hallucinated data (claims not verifiable from the raw output).
- Malformed JSON or broken structured data.
- Contradictions with existing `specification/` content.

If the sentinel flags the output, discard it and fall back to local reasoning. Do not patch or guess at malformed data.
