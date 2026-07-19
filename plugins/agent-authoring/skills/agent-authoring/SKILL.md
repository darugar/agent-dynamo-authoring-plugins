---
name: agent-authoring
description: Author, update, or run agents/work queues on the Agent Dynamo platform (AgentSpec YAML, workflow steps, dataflow between steps, sub-agents, sandboxed custom tools). Use whenever writing or editing an AgentSpec, designing a workflow agent, creating a custom tool for an agent, or debugging an agent run.
---

# Authoring Agent Dynamo agents

Agent Dynamo is a hosted platform for LLM agents: single-prompt ("agentic")
agents, multi-step workflow agents, sub-agents, and work queues. Everything is
authored as AgentSpec YAML and managed through the `agent-dynamo` MCP server —
nothing to install or run yourself. Agents execute on the platform, not on this
machine.

## Connect (once)

With the **agent-authoring** plugin installed, the `agent-dynamo` MCP server is
already registered. Export your key and restart the harness:

```bash
export AGENTDYNAMO_API_KEY=ad_...
```

Without the plugin, register the server by hand as a streamable-HTTP server:

```bash
# Claude Code
claude mcp add --transport http agent-dynamo https://app.agentdynamo.com/mcp \
  --header "Authorization: Bearer ad_..."

# Codex
codex mcp add agent-dynamo --url https://app.agentdynamo.com/mcp \
  --bearer-token-env-var AGENTDYNAMO_API_KEY
```

Mint the `ad_...` key in the Agent Dynamo web app under **Settings → API
keys**. Every tool call runs as you, scoped to your account. Any other MCP
client takes the same URL (`/mcp` exactly — no trailing slash) with an
`Authorization: Bearer ad_...` header.

## Authoring

1. **Call the `authoring_guide` tool first** — it is the canonical reference
   for AgentSpec fields, workflow execution semantics (step ordering,
   prompt/output flow between steps, artifacts, sub-agent mechanisms), work-queue specs,
   and the full tool loop. Do not author from assumptions.
2. Call `building_blocks` before writing any spec — never guess model ids,
   tool names, or MCP server names. `spec_schema` returns the JSON Schemas
   for both spec kinds (`Agent`, `WorkQueue`).
3. Iterate: `validate_spec` (structure) → `apply_spec(dry_run=True)`
   (reference resolution) → `apply_spec` (same slug = new version) →
   `run_agent(slug, prompt, wait=True)` → `execution_log(execution_id)` to
   inspect step-by-step behavior.

## Sandboxed dynamic tools (custom tools)

When an agent needs a capability no native tool or MCP server provides and
it's a pure computation (parse/transform/extract — no network or disk), write
a **sandboxed tool**: a small Python program run in a secure sandbox, stored
and versioned per account.

1. **Call the `sandboxed_tool_guide` tool first** — the sandbox is a
   restricted Python subset with a script-style contract (parameters arrive
   as variables, the last expression is the result). Do not write tool code
   from assumptions.
2. Iterate: `validate_sandboxed_tool` (parse check) → `test_sandboxed_tool`
   (your test cases run in the real sandbox) → `upsert_sandboxed_tool`
   (each save is a new immutable version) → `publish_sandboxed_tool`
   (agents only ever run the published version; publishing an older version
   number rolls back).
3. Reference the published tool from a spec via `sandboxed_tools: [name]`
   (listed in `building_blocks` too).

Tools needing HTTP calls are not sandboxed-tool material — give the agent
the `http_request` native tool or an MCP server instead.

## Gotchas

- `run_agent` returns two ids: `run_id` (for `execution_status`) and
  `execution_id` (for `execution_log`).
- `spec_schema`, `validate_spec`, and `authoring_guide` work without a key;
  everything else requires your Bearer key.
- Older specs used `memory_inputs`/`memory_output` on steps; those fields are
  removed and rejected with a migration hint. Use `context_from: [step-ids]`
  (the step receives those outputs in context) and `emit: last_tool_result`
  (the step outputs its raw tool result) instead. Don't narrate mem://
  mechanics in step instructions — the platform injects that guidance.
