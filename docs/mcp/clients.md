# Connect your AI tool

The InflowCRM MCP server works with any AI tool that supports **remote MCP servers over Streamable HTTP**, either
by **signing in with your CRM account (OAuth)** or with **an API key in a header**. Everything you need:

| Setting | Value |
|---|---|
| URL | `https://mcp.inflowcrm.pl` |
| Transport | Streamable HTTP (sometimes called "HTTP" or "streamable-http"; **not** SSE) |
| Authentication | OAuth — sign in on your company's CRM page (no key needed), **or** |
| Header | `Authorization: Bearer YOUR_API_KEY` (or `x-api-key: YOUR_API_KEY`) |

MCP must be enabled for your company first; for the API key route you also need an integration user's key — see
[MCP Server → Quick start](README.md). Signing in works for company administrators, and for employees only when
your company has asked InflowCRM to allow them ([details](README.md#who-can-sign-in)).

> Keep the key out of files you commit. Where a tool can read it from an environment variable, use
> `INFLOWCRM_API_KEY`.

**Jump to:** [claude.ai and Claude Desktop](#claudeai-and-claude-desktop-connector) · [ChatGPT](#chatgpt) · [Gemini](#gemini) ·
[Universal setup prompt](#universal-setup-prompt) · [Claude Code](#claude-code) ·
[Claude Desktop with an API key](#claude-desktop-with-an-api-key) · [OpenAI Codex CLI](#openai-codex-cli) ·
[Cursor](#cursor) · [VS Code](#vs-code-github-copilot) · [n8n](#n8n) ·
[Python](#python-mcp-sdk) · [OpenAI Agents SDK](#openai-agents-sdk) · [LangChain / LangGraph](#langchain-langgraph)

---

## claude.ai and Claude Desktop (connector)

claude.ai and the Claude Desktop app share the same **custom connectors**. Connecting signs you in with your CRM
account — no API key.

1. Open **Settings → Connectors** and choose **Add custom connector**. (In a Team / Enterprise organization an owner
   may have to add the connector for the organization first.)
2. Name it `InflowCRM` and enter the URL `https://mcp.inflowcrm.pl`. Leave the advanced OAuth fields empty — the app
   registers itself.
3. Click **Connect**. A page *"Połącz aplikację AI z InflowCRM"* asks for **your CRM address** — the one you log in
   at, e.g. `firma.inflowstore.pl` — then **Dalej**.
4. Log in to your CRM if asked. The consent screen shows the app, your account and your company — check them and
   **allow** access.
5. Back in Claude, enable the InflowCRM connector in a chat's tools menu and ask e.g. *"How many customers do we
   have?"*.

**Recommended:** in the connector's tool settings set the tools that create, change or delete data to **needs
approval** (ask every time). This is the confirmation that cannot be bypassed by text inside CRM data. See
[Confirmations](README.md#confirmations) for the tools that also require `confirm: true`.

The assistant works with **your rights** in the CRM. When InflowCRM enables more tools for your company, disconnect
and connect again (or start a new chat) to see them.

Skill: if your plan lets you add skills (Settings → Capabilities), upload a ZIP of the
[`skills/inflowcrm-mcp`](https://github.com/InflowCRM/public-api-docs/tree/main/skills/inflowcrm-mcp) folder;
otherwise paste the [skill text](skill.md#copy-and-paste) into a Project's instructions.

| Message | What to do |
|---|---|
| *Nie znaleziono CRM o tym adresie albo MCP nie jest w nim włączone.* | Check the CRM address; ask InflowCRM to enable MCP for your company |
| *To konto nie może łączyć aplikacji AI z CRM. Skontaktuj się z administratorem firmy.* | Your account may not connect AI apps — ask your company administrator (employees need the feature enabled for the company) |
| *Ten link wygasł. Wróć do aplikacji i połącz się jeszcze raz.* | The sign-in link is valid for 10 minutes — click Connect again |

---

## ChatGPT

ChatGPT connects to remote MCP servers as **custom MCP plugins** with OAuth. You add one in ChatGPT on the web.
Availability depends on your ChatGPT plan, and workspace permissions or Lockdown may block custom servers — ask your
workspace admin if you do not see the option.

1. Open [ChatGPT Plugins](https://chatgpt.com/plugins), click **+** and choose **Add custom MCP server**.
2. Name it `InflowCRM`. Under **Connection** enter the server URL `https://mcp.inflowcrm.pl`.
3. Choose **OAuth** as the authentication and leave any client credentials empty — ChatGPT registers itself.
4. Review the risk warning, choose **I understand and want to continue**, then **Create as a plugin**.
5. Find the plugin under your personal plugins (or the workspace where you created it) and install it. Sign in exactly
   as in steps 3–4 for [claude.ai](#claudeai-and-claude-desktop-connector): your CRM address, log in, allow.
6. In a chat, type `@` and pick the plugin, then ask e.g. *"How many customers do we have?"*.

ChatGPT asks for confirmation before write actions. Review what the tool is about to send, and keep the "remember"
option off for tools that change or delete data. When InflowCRM enables more tools for your company, open the app's
details page in the app settings and refresh it; you can also switch individual tools off there.

Skill: paste the [skill text](skill.md#copy-and-paste) into the project's or custom GPT's instructions.

---

## Gemini

### Gemini CLI

Sign in with your CRM account — add the server without a header, then start `gemini` and run `/mcp auth inflowcrm`:

```bash
gemini mcp add --scope user --transport http inflowcrm https://mcp.inflowcrm.pl
```

Or use an API key:

```bash
gemini mcp add --scope user --transport http inflowcrm https://mcp.inflowcrm.pl \
  --header "Authorization: Bearer YOUR_API_KEY"
```

Check with `gemini mcp list`, or with `/mcp` inside Gemini CLI — before you sign in the server shows as disconnected.
Do not add `--trust`: it switches off the confirmations for every tool. Skill: add the
[skill text](skill.md#copy-and-paste) to `~/.gemini/GEMINI.md`.

### Gemini app

Google currently offers custom apps only in **English**, to users **aged 18+ in the US**, signed in with a **personal
Google account** (not work or school), and with **Keep Activity** on. If that rules you out — for example you are in
Poland — use Gemini CLI, or another tool from this page.

1. Go to [gemini.google.com](https://gemini.google.com) on a computer (custom apps can only be connected on the web;
   the mobile app can use them afterwards).
2. Open **Settings → Connected Apps** and, under **Custom apps**, choose **Add a custom app**.
3. Enter `https://mcp.inflowcrm.pl` and click **Next**. Leave the advanced fields empty — Gemini registers itself.
4. Sign in as in steps 3–4 for [claude.ai](#claudeai-and-claude-desktop-connector): your CRM address, log in, allow.
5. In a prompt, type `@` and pick the app.

Gemini asks for confirmation before write actions. Skill: paste the [skill text](skill.md#copy-and-paste) into your
prompt or a Gem's instructions.

### Gemini Enterprise

The custom MCP connector in Gemini Enterprise needs an OAuth client registered in advance, which is not covered by the
sign-in flow above. [Contact us](../contact.md) if your company wants to connect it.

---

## Universal setup prompt

Not sure how your tool is configured? Paste this into your AI agent (Claude Code, Codex, Gemini CLI, Cursor
agent…) and let it set itself up. You do not need to edit it — the agent asks you for the key.

```text
Set up the InflowCRM MCP server and the InflowCRM agent skill for the tool you are running in.

Connection:
- URL: https://mcp.inflowcrm.pl
- Transport: Streamable HTTP (remote MCP server, not stdio, not SSE)
- Authentication, one of:
  a) OAuth: the server supports MCP OAuth 2.1 (dynamic client registration, PKCE); I sign in on my
     company's CRM page, no key needed;
  b) API key header: Authorization: Bearer <API key>
- Server name: inflowcrm

Steps:
1. Work out which AI tool/client you are running in and how it registers remote MCP servers
   (CLI command or config file). If it can do neither MCP OAuth nor custom headers on a remote HTTP
   server, tell me and stop. If OAuth sign-in fails in your tool, fall back to an API key.
2. Ask me whether I want to sign in with my CRM account (OAuth, if your tool supports it) or use an
   API key. For OAuth, register the server without any auth header and tell me how to start the
   sign-in (I will enter my CRM address, log in and allow access). For an API key, ask me for it;
   prefer a reference to the environment variable INFLOWCRM_API_KEY over writing the key into a file;
   never put the key into a file tracked by git, and never echo it back.
3. Register the server under the name "inflowcrm" at user/global level unless I ask for this project only.
   If your tool can require approval per tool, set the InflowCRM tools that create, update or delete
   data (records, templates, fields, list views, roles) to "ask every time".
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

Or sign in with your CRM account instead of a key: add the server without `--header`, run `/mcp`, select
`inflowcrm` and choose **Authenticate** — the browser opens the sign-in page.

```bash
claude mcp add --transport http --scope user inflowcrm https://mcp.inflowcrm.pl
```

Skill: `/plugin marketplace add InflowCRM/public-api-docs` then `/plugin install inflowcrm@inflowcrm`
([details](skill.md)). Check the connection with `/mcp`.

---

## Claude Desktop with an API key

The easiest way is the [connector with sign-in](#claudeai-and-claude-desktop-connector). Connectors cannot send an
API-key header, so to connect Claude Desktop **with an integration user's API key** use its local configuration
with the [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) bridge (requires [Node.js](https://nodejs.org) 18+):

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
- **Signing in as an integration user** — integration users connect with their API key only.
