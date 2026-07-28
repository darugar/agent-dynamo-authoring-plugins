# Agent Dynamo authoring plugins

Coding-harness plugins for the [Agent Dynamo](https://app.agentdynamo.com)
platform. Currently one plugin: **agent-authoring** — author, run, and debug
agents, workflow agents, and work queues directly from your harness, with zero
local install.

Supported harnesses: **Claude Code** and **Codex**. One marketplace, one
plugin, one shared skill; each harness reads its own manifest.

## Install

### Claude Code

```shell
/plugin marketplace add darugar/agent-dynamo-authoring-plugins
/plugin install agent-authoring@agent-dynamo
```

### Codex

```bash
codex plugin marketplace add darugar/agent-dynamo-authoring-plugins
codex plugin add agent-authoring@agent-dynamo
```

Then set your API key in the environment the harness runs in (mint one in the
Agent Dynamo web app under **Settings → API keys**):

```bash
export AGENTDYNAMO_API_KEY=ad_...
```

Add that line to your shell profile (`~/.zshrc` / `~/.bashrc`) so it survives
new terminals, and restart the harness. That's it — the plugin bundles the
connection to the hosted `agent-dynamo` MCP server, so its tools appear
automatically, along with an `agent-authoring` skill that guides spec writing.

## What you get

- **MCP connection** to `https://app.agentdynamo.com/mcp`, authenticated with
  your key — every tool call is scoped to your account. Tools include
  `building_blocks`, `validate_spec`, `apply_spec`, `run_agent`,
  `execution_log`, `enqueue_items`, `queue_status`, `queue_control`,
  `export_agent`, and `authoring_guide` (the full authoring reference,
  fetched on demand — no docs to install).
- **Skill** (`agent-authoring`) that tells the model how to author AgentSpecs:
  consult the guide first, look up real building-block names, then validate →
  dry-run → apply → run → inspect.

Try: *"Create an agent that summarizes a URL I give it, then run it on
example.com"*.

## Without the plugin

Any MCP client can connect directly — same endpoint, same key:

```bash
# Claude Code
claude mcp add --transport http agent-dynamo https://app.agentdynamo.com/mcp \
  --header "Authorization: Bearer ad_..."

# Codex
codex mcp add agent-dynamo --url https://app.agentdynamo.com/mcp \
  --bearer-token-env-var AGENTDYNAMO_API_KEY
```

Other clients (Cursor, claude.ai custom connectors) take the same URL — `/mcp`
exactly, no trailing slash — with an `Authorization: Bearer ad_...` header.

## Headless & CI use

**Claude Code.** Interactive sessions prompt for tool permissions the first
time — nothing to configure. But non-interactive runs (`claude -p`, the Agent
SDK, CI) and shared permission allowlists in `.claude/settings.json` must name
the tools explicitly, and plugin-bundled MCP servers get a namespaced
permission scope (`plugin_<plugin>_<server>`), not the server name alone:

```bash
claude -p "List my Agent Dynamo agents" \
  --allowedTools "mcp__plugin_agent-authoring_agent-dynamo__*"
```

Two gotchas: a bare `mcp__*` wildcard is rejected (allow rules must name a
server scope; globs are only valid in the tool position), and allowlists
written for a manually added server (`mcp__agent-dynamo__*` via
`claude mcp add`) do not match the plugin's tools. Hook matchers follow the
same scoped naming.

**Codex.** `codex exec` prompts for MCP tool approval and cancels the call if
nothing answers — `approval_policy = "never"` does not auto-approve it. Expect
`user cancelled MCP tool call` in unattended runs unless you explicitly relax
approvals for the session.

## Troubleshooting

- **"Missing Authorization header"** — `AGENTDYNAMO_API_KEY` isn't set in the
  environment the harness started from. Set it and restart the harness.
- **"Authentication failed (403)"** — the key is wrong, revoked, or belongs to
  a different instance (a key minted against a local dev instance will 403
  against `app.agentdynamo.com`). Mint a new one under Settings → API keys.

## Layout

```
.claude-plugin/marketplace.json          Claude Code marketplace catalog
.agents/plugins/marketplace.json         Codex marketplace catalog
plugins/agent-authoring/
  .claude-plugin/plugin.json             Claude Code manifest
  .codex-plugin/plugin.json              Codex manifest
  .mcp.json                              MCP server, Claude Code shape
  .mcp.codex.json                        MCP server, Codex shape
  skills/agent-authoring/SKILL.md        shared by both harnesses
```

The two MCP files are not interchangeable: Claude Code uses
`{"type": "http", "headers": {"Authorization": "Bearer ${VAR}"}}`, while Codex
uses `{"url": ..., "bearer_token_env_var": "VAR"}`. The Codex manifest points
at its own file via `"mcpServers": "./.mcp.codex.json"`.

## Maintainers

**This repo owns the skill.** There is no copy in the app repo and nothing to
sync. That works because the skill deliberately holds only stable procedure —
"call `authoring_guide` first, look up real building-block names, then validate
→ dry-run → apply → run → inspect" — while every version-specific detail
reaches the model at runtime through the MCP tools the platform serves:
`authoring_guide` (the full authoring reference), `building_blocks` (real model
ids and tool names), `spec_schema` (live JSON Schema), and `validate_spec`'s
error hints (including migration hints for removed spec fields).

So a platform change usually needs **no release here**. Edit this skill only
when the authoring *procedure* changes — a new step in the loop, a renamed
tool, a changed connection story.

To release: bump `version` in **both**
`plugins/agent-authoring/.claude-plugin/plugin.json` and
`plugins/agent-authoring/.codex-plugin/plugin.json` — users only see updates on
a version bump — then commit and push. Validate before pushing:

```bash
claude plugin validate .
codex plugin marketplace add .   # then: codex plugin add agent-authoring@agent-dynamo
```
