# Connect your AI tool

The InflowCRM MCP server works with any AI tool that supports **remote MCP servers over Streamable HTTP with a
custom header**. Everything you need:

| Setting | Value |
|---|---|
| URL | `https://mcp.inflowcrm.pl` |
| Transport | Streamable HTTP (sometimes called "HTTP" or "streamable-http"; **not** SSE) |
| Header | `Authorization: Bearer YOUR_API_KEY` (or `x-api-key: YOUR_API_KEY`) |

Get the API key and enable MCP first — see [MCP Server → Quick start](README.md).

> Keep the key out of files you commit. Where a tool can read it from an environment variable, use
> `INFLOWCRM_API_KEY`.

**Jump to:** [Universal setup prompt](#universal-setup-prompt) · [Claude Code](#claude-code) ·
[Claude Desktop](#claude-desktop) · [OpenAI Codex CLI](#openai-codex-cli) · [Gemini CLI](#gemini-cli) ·
[Cursor](#cursor) · [VS Code](#vs-code-github-copilot) · [Windsurf](#windsurf) · [n8n](#n8n) ·
[Python](#python-mcp-sdk) · [OpenAI Agents SDK](#openai-agents-sdk) · [LangChain / LangGraph](#langchain-langgraph)

---

## Universal setup prompt

Not sure how your tool is configured? Paste this into your AI agent (Claude Code, Codex, Gemini CLI, Cursor
agent…) and let it set itself up. You do not need to edit it — the agent asks you for the key.

```text
Set up the InflowCRM MCP server and the InflowCRM agent skill for the tool you are running in.

Connection:
- URL: https://mcp.inflowcrm.pl
- Transport: Streamable HTTP (remote MCP server, not stdio, not SSE)
- Auth header: Authorization: Bearer <API key>
- Server name: inflowcrm

Steps:
1. Work out which AI tool/client you are running in and how it registers remote MCP servers
   (CLI command or config file). If it cannot connect to a remote HTTP server with custom headers,
   tell me and stop.
2. Ask me for my InflowCRM API key. Prefer a reference to the environment variable INFLOWCRM_API_KEY
   over writing the key into a file; never put the key into a file tracked by git, and never echo it back.
3. Register the server under the name "inflowcrm" at user/global level unless I ask for this project only.
4. Install the agent skill:
   https://raw.githubusercontent.com/InflowCRM/public-api-docs/main/skills/inflowcrm-mcp/SKILL.md
   - If your tool supports Agent Skills, save it as inflowcrm-mcp/SKILL.md in your skills directory
     (Claude Code: ~/.claude/skills/inflowcrm-mcp/SKILL.md). Optionally also save
     https://raw.githubusercontent.com/InflowCRM/public-api-docs/main/skills/inflowcrm-mcp/references/tools.md
     as inflowcrm-mcp/references/tools.md.
   - Otherwise add its content to your persistent instructions (e.g. AGENTS.md, GEMINI.md, project rules).
5. Tell me if I need to restart or reload the tool, then verify: call the inflowcrm tool list_modules
   and show me the module names. Do not create, change or delete any CRM data during setup.
6. Report what you changed (files, commands) and how to undo it.
```

---

## Claude Code

```bash
claude mcp add --transport http --scope user inflowcrm https://mcp.inflowcrm.pl \
  --header "Authorization: Bearer YOUR_API_KEY"
```

Skill: `/plugin marketplace add InflowCRM/public-api-docs` then `/plugin install inflowcrm@inflowcrm`
([details](skill.md)). Check the connection with `/mcp`.

---

## Claude Desktop

Claude Desktop's **custom connectors** (Settings → Connectors) cannot send an API-key header. Use its local
configuration with the [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) bridge instead (requires
[Node.js](https://nodejs.org) 18+):

1. **Settings → Developer → Edit Config** opens `claude_desktop_config.json`.
2. Add the server:

```json
{
  "mcpServers": {
    "inflowcrm": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://mcp.inflowcrm.pl",
        "--header",
        "Authorization:${INFLOWCRM_AUTH}"
      ],
      "env": {
        "INFLOWCRM_AUTH": "Bearer YOUR_API_KEY"
      }
    }
  }
}
```

3. Restart Claude Desktop. The InflowCRM tools appear in the chat's tools menu.

The header is passed through `env` because some systems mangle spaces in arguments. Skill: if your plan lets you add
skills (Settings → Capabilities), upload a ZIP of the
[`skills/inflowcrm-mcp`](https://github.com/InflowCRM/public-api-docs/tree/main/skills/inflowcrm-mcp) folder;
otherwise paste the [skill text](skill.md#copy-and-paste) into a Project's instructions.

---

## OpenAI Codex CLI

```bash
export INFLOWCRM_API_KEY=YOUR_API_KEY        # add to your shell profile
codex mcp add inflowcrm --url https://mcp.inflowcrm.pl --bearer-token-env-var INFLOWCRM_API_KEY
```

Equivalent `~/.codex/config.toml`:

```toml
[mcp_servers.inflowcrm]
url = "https://mcp.inflowcrm.pl"
bearer_token_env_var = "INFLOWCRM_API_KEY"
```

Skill: add the [skill text](skill.md#copy-and-paste) to `~/.codex/AGENTS.md` (global) or `AGENTS.md` in a project.

---

## Gemini CLI

```bash
gemini mcp add --scope user --transport http inflowcrm https://mcp.inflowcrm.pl \
  --header "Authorization: Bearer YOUR_API_KEY"
```

Check with `gemini mcp list`. Skill: add the [skill text](skill.md#copy-and-paste) to `~/.gemini/GEMINI.md`.

---

## Cursor

*Config example* — `~/.cursor/mcp.json` (global) or `.cursor/mcp.json` (project):

```json
{
  "mcpServers": {
    "inflowcrm": {
      "url": "https://mcp.inflowcrm.pl",
      "headers": { "Authorization": "Bearer YOUR_API_KEY" }
    }
  }
}
```

Skill: save the [skill text](skill.md#copy-and-paste) as a rule in `.cursor/rules/inflowcrm.mdc`.

---

## VS Code (GitHub Copilot)

*Config example* — `.vscode/mcp.json` (VS Code prompts for the key and stores it securely):

```json
{
  "inputs": [
    { "type": "promptString", "id": "inflowcrm-key", "description": "InflowCRM API key", "password": true }
  ],
  "servers": {
    "inflowcrm": {
      "type": "http",
      "url": "https://mcp.inflowcrm.pl",
      "headers": { "Authorization": "Bearer ${input:inflowcrm-key}" }
    }
  }
}
```

Use the tools in Copilot Chat's **Agent** mode. Skill: add the [skill text](skill.md#copy-and-paste) to
`.github/copilot-instructions.md`.

---

## Windsurf

*Config example* — `~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "inflowcrm": {
      "serverUrl": "https://mcp.inflowcrm.pl",
      "headers": { "Authorization": "Bearer YOUR_API_KEY" }
    }
  }
}
```

---

## n8n

*Config example* — use the **MCP Client Tool** node attached to an **AI Agent** node:

- **Endpoint:** `https://mcp.inflowcrm.pl`
- **Server Transport:** HTTP Streamable
- **Authentication:** Bearer Auth (credential with your API key) — or Header Auth with `x-api-key`
- **Tools to include:** pick the tools the workflow needs (e.g. only read tools)

For classic, deterministic workflows the [InflowCRM n8n node](../n8n-node.md) (REST API) is usually the better fit.

---

## Python (MCP SDK)

Tested with the official `mcp` package 2.x:

```python
import asyncio, os
from mcp import ClientSession
from mcp.client.streamable_http import create_mcp_http_client, streamable_http_client

async def main():
    http = create_mcp_http_client(headers={"Authorization": f"Bearer {os.environ['INFLOWCRM_API_KEY']}"})
    async with http, streamable_http_client("https://mcp.inflowcrm.pl", http_client=http) as streams:
        async with ClientSession(streams[0], streams[1]) as session:
            await session.initialize()
            tools = await session.list_tools()
            print([t.name for t in tools.tools])
            result = await session.call_tool("count_records", {"module": "customer"})
            print(result.content[0].text)

asyncio.run(main())
```

With the 1.x SDK use `streamablehttp_client("https://mcp.inflowcrm.pl", headers={...})` instead.

---

## OpenAI Agents SDK

*Example* (`openai-agents` package):

```python
import asyncio, os
from agents import Agent, Runner
from agents.mcp import MCPServerStreamableHttp

async def main():
    async with MCPServerStreamableHttp(
        name="inflowcrm",
        params={
            "url": "https://mcp.inflowcrm.pl",
            "headers": {"Authorization": f"Bearer {os.environ['INFLOWCRM_API_KEY']}"},
        },
    ) as crm:
        agent = Agent(name="CRM assistant", instructions=open("SKILL.md").read(), mcp_servers=[crm])
        result = await Runner.run(agent, "How many customers do we have?")
        print(result.final_output)

asyncio.run(main())
```

The skill file doubles as the agent's instructions.

---

## LangChain / LangGraph

*Example* (`langchain-mcp-adapters` package):

```python
import os
from langchain_mcp_adapters.client import MultiServerMCPClient

client = MultiServerMCPClient({
    "inflowcrm": {
        "transport": "streamable_http",
        "url": "https://mcp.inflowcrm.pl",
        "headers": {"Authorization": f"Bearer {os.environ['INFLOWCRM_API_KEY']}"},
    }
})
tools = await client.get_tools()   # pass to create_react_agent(...) or your own agent
```

---

## Not supported

- **SSE-only clients** — the server speaks Streamable HTTP only.
- **Browser-based clients** calling the server directly from a web page — browser `Origin` headers are rejected.
- **Connectors that require OAuth** (e.g. claude.ai / ChatGPT custom connectors without header support) — the
  server authenticates with an API key; OAuth is not available yet.
