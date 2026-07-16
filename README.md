# Agent Dynamo plugins for Claude Code

Claude Code plugins for the [Agent Dynamo](https://app.agentdynamo.com)
platform. Currently one plugin: **agent-authoring** — author, run, and debug
agents, workflow agents, and work queues directly from Claude Code, with zero
local install.

## Install

```shell
/plugin marketplace add darugar/agent-dynamo-claude-plugins
/plugin install agent-authoring@agent-dynamo
```

Then set your API key in the environment Claude Code runs in (mint one in the
Agent Dynamo web app under **Settings → API keys**):

```bash
export AGENTDYNAMO_API_KEY=ad_...
```

Add that line to your shell profile (`~/.zshrc` / `~/.bashrc`) so it survives
new terminals, and restart Claude Code. That's it — the plugin bundles the
connection to the hosted `agent-dynamo` MCP server, so its tools appear
automatically, along with an `agent-authoring` skill that guides spec writing.

## What you get

- **MCP connection** to `https://app.agentdynamo.com/mcp`, authenticated with
  your key — every tool call is scoped to your account. Tools include
  `building_blocks`, `validate_spec`, `apply_spec`, `run_agent`,
  `execution_log`, `enqueue_items`, `queue_status`, `queue_control`,
  `export_agent`, and `authoring_guide` (the full authoring reference,
  fetched on demand — no docs to install).
- **Skill** (`/agent-authoring:agent-authoring`) that tells Claude how to
  author AgentSpecs: consult the guide first, look up real building-block
  names, then validate → dry-run → apply → run → inspect.

Try: *"Create an agent that summarizes a URL I give it, then run it on
example.com"*.

## Troubleshooting

- **"Missing Authorization header"** — `AGENTDYNAMO_API_KEY` isn't set in the
  environment Claude Code started from. Set it and restart Claude Code.
- **"Authentication failed (403)"** — the key is wrong or revoked; mint a new
  one under Settings → API keys.
- Other MCP clients (Cursor, claude.ai custom connectors) don't use plugins;
  connect them directly to `https://app.agentdynamo.com/mcp` (no trailing
  slash) with the same `Authorization: Bearer ad_...` header.

## Maintainers

The skill's source of truth is `.claude/skills/agent-authoring/SKILL.md` in
the app repo; `just publish-skill` copies it here. To release: bump `version`
in `plugins/agent-authoring/.claude-plugin/plugin.json` (users only see
updates on a version bump), commit, push. Validate before pushing:

```bash
claude plugin validate .
```
