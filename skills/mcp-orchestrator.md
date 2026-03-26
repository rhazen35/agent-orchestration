---
name: mcp-orchestrator
description: Dynamically discovers and utilizes available MCP tools to fulfill requirements that cannot be solved by local file editing alone. Acts as the bridge between static specs and live environmental capabilities. Use at the start of the Design phase or when a task requires external data.
---

Discover what tools are available in the current environment and use them purposefully to enrich the spec and implementation — without wasting tokens or introducing unverified data.

## Trigger

- Start of Phase 1 (Design) — after `discovery`, before `grill-me`. Runs in full capability mapping mode.
- Phase 3 (Production) — invoked by `task-processor` in lightweight mode when a task requires external data (API response, live schema, library version, etc.).
- When a task in `inbox/{namespace}/tasks.md` explicitly references external data that is not present in `specification/` or `archive/`.
- When the `spec-architect` identifies a requirement that local files cannot answer.

## Modes

### Full Mode (Design Phase)
Runs complete capability mapping via `list_tools`, matches all tools against pending tasks, and enriches the spec with live data. Used by `spec-architect`.

### Lightweight Mode (Production Phase)
Skips full capability mapping. The worker states a specific need, the orchestrator checks cache first, and only calls a tool if the data is missing or stale. Used by `task-processor`. Respects JIT context — no broad exploration.

## Execution Protocol

### Step 1: Environmental Awareness

At the start of every session, call the MCP-native `list_tools` function to enumerate all available tools in the current environment.

Match the returned tool schemas against `steering/mcp-policy.md` to determine which tools are authorized for the current task. Tools outside the defined capability categories require explicit human approval before use.

Compare the authorized tool list against the current `inbox/{namespace}/tasks.md` to identify which tools are relevant to pending work. Apply the Local-First rule from `steering/mcp-etiquette.md` — check `specification/`, `archive/`, and `cache/` before deciding a tool call is necessary.

For each relevant authorized tool found, record:
- Tool name
- Capability category (search / filesystem / runtime / database / version control)
- Relevance to current tasks

Examples of tool-to-task matching:
- `brave_search` available → use to verify library versions instead of guessing
- `fetch_url` available → retrieve API docs referenced in the spec
- `github` available → read issues or PRs linked to requirements
- `postgres` available → inspect live schema instead of assuming structure
- No relevant tools → proceed with local files only

Persist the capability map to `cache/discovery-report.md`.

### Step 2: Justification (Micro-Plan)

Before calling any external tool, internalize a micro-plan:

```
Tool: [tool name]
Reason: [why this tool is needed for the current task]
Cost-Check: [is this information already in cache/discovery-report.md or archive/mcp-data/?]
```

If the information already exists in `archive/mcp-data/{namespace}/`, use the cached result. Do not re-call the tool.

### Step 3: Execution & Persistence

Execute the tool call.

- Summarize the findings (max 2,000 tokens per `steering/mcp-governance.md`).
- Persist the full output to `archive/mcp-data/{namespace}/{tool-name}-{YYYY-MM-DD}.md`.
- Inject only the summary into the current task context.

This ensures the Worker can reference findings without re-calling the tool.

### Step 4: Sentinel Filter

Before using any MCP output to update `specification/` or any spec file, pass it through the `sentinel`:

- Flag hallucinated data (claims not verifiable from the raw output).
- Flag malformed JSON or broken structured data.
- Flag contradictions with existing `specification/` content.

If flagged: discard the output, fall back to local reasoning, do not patch or guess.

## Constraints

- Never call a write/delete/push tool without a sentinel pre-check or human `[CONTINUE]`.
- Never inject raw tool output directly into spec files.
- If a tool fails twice, mark the task `[BLOCKED]` per `steering/mcp-governance.md`.
- Always prefer cached results in `archive/mcp-data/` over re-calling a tool.

## Project Overrides

This skill can be extended or replaced at the project level. Create `.agent/skills/mcp-orchestrator.md` in your project root to override this default. See `steering/overrides.md` for the resolution order and extension pattern.
