---
name: agent-authoring
description: Author, update, or run agents/work queues on the Agent Dynamo platform (AgentSpec YAML, workflow steps, dataflow between steps, sub-agents). Use whenever writing or editing an AgentSpec, designing a workflow agent, or debugging an agent run.
---

# Authoring Agent Dynamo agents

Agent Dynamo is a hosted platform for LLM agents: single-prompt ("agentic")
agents, multi-step workflow agents, sub-agents, and work queues. Everything is
authored as AgentSpec YAML and managed through the `agent-dynamo` MCP server —
no local install or repo checkout needed.

## Connect (once)

```bash
claude mcp add --transport http agent-dynamo https://app.agentdynamo.com/mcp \
  --header "Authorization: Bearer ad_..."
```

Mint the `ad_...` key in the Agent Dynamo web app under **Settings → API
keys**. Use `/mcp` exactly — no trailing slash. Every tool call runs as you,
scoped to your account. (Other MCP clients — Cursor, claude.ai custom
connectors — take the same URL + header.)

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
