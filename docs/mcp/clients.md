# Connect your AI tool

The InflowCRM MCP server works with AI tools that can connect to a **remote MCP server**. You connect either by
**signing in with your CRM account** or, in developer tools, with **an API key**.

| Setting | Value |
|---|---|
| URL | `https://mcp.inflowcrm.pl` |
| Transport | Streamable HTTP (sometimes called "HTTP" or "streamable-http"; **not** SSE) |
| Sign-in | OAuth: sign in on your company's CRM page, no key needed |
| API key | `Authorization: Bearer YOUR_API_KEY` (or `x-api-key: YOUR_API_KEY`) |

MCP must be enabled for your company first; for the API key route you also need an integration user's key — see
[MCP Server → Quick start](README.md).

> **For now MCP is available to company administrators only.** Employees cannot connect yet. We will soon add an option that lets employees use MCP within the permissions of their own CRM role.

> Keep the key out of files you commit. Where a tool can read it from an environment variable, use
> `INFLOWCRM_API_KEY`.

**Not a developer?** Use [Claude](#claude-desktop-and-claudeai), [ChatGPT](#chatgpt), [Gemini](#gemini),
[Perplexity](#perplexity) or [Grok](#grok) and sign in with your CRM account. You need no terminal and no key.

**Something not working?** See [Troubleshooting](troubleshooting.md).

**Jump to:** [Claude Desktop and claude.ai](#claude-desktop-and-claudeai) · [Claude Code](#claude-code) ·
[ChatGPT](#chatgpt) · [Gemini](#gemini) · [Perplexity](#perplexity) · [Grok](#grok) ·
[Limit what each person can do](#limit-what-each-person-can-do) · [Universal setup prompt](#universal-setup-prompt) ·
[OpenAI Codex CLI](#openai-codex-cli) · [Cursor](#cursor) · [VS Code](#vs-code-github-copilot) · [n8n](#n8n)

---

## Claude Desktop and claude.ai

The Claude Desktop app and claude.ai share the same **custom connectors**, so you add InflowCRM once. Custom connectors
work on every Claude plan; the **Free plan allows one**.

1. Open **Customize → Connectors**, click **+ Add**, then **Add custom connector**.
   - **Team / Enterprise:** an owner first adds the connector in **Organization settings → Connectors**. A company
     administrator then finds it in **Customize → Connectors**, clicks **Connect** and signs in with their CRM account.
2. Name it `InflowCRM`, enter the URL `https://mcp.inflowcrm.pl` and click **Continue**. Review the detected settings
   and click **Continue** again.
3. Under **Authentication** choose **Sign in now**. Leave **OAuth client** as it is and **Request headers** empty,
   then click **Add**.
4. On the page *"Połącz aplikację AI z InflowCRM"* enter **your CRM address** — the one you log in at, e.g.
   `firma.inflowstore.pl` — then **Dalej**.
5. Log in to your CRM if asked. The consent screen shows the app, your account and your company — check them and
   **allow** access.
6. In a chat, turn the connector on with **+ → Connectors** and ask e.g. *"How many customers do we have?"*.

The assistant works with **your rights** in the CRM. When InflowCRM enables more tools for your company, disconnect
and connect again (or start a new chat) to see them.

**Stay in control of changes:**

- When Claude asks to use a tool that changes data, read what it wants to send. Do not choose **Allow always** for
  tools that create, change or delete data.
- In the **Search and tools** menu you can switch off individual tools, for example every tool that writes.
- During **Research**, Claude can use connector tools without asking again, so switch off write tools first.

See also [Confirmations](README.md#confirmations) for the tools that also require `confirm: true`.

**Skill (optional):** if your plan lets you add skills, upload a ZIP of the
[`skills/inflowcrm-mcp`](https://github.com/InflowCRM/public-api-docs/tree/main/skills/inflowcrm-mcp) folder;
otherwise paste the [skill text](skill.md#copy-and-paste) into a Project's instructions.

**API key instead of signing in:** the connector form has a **Request headers** field, where you can add
`Authorization: Bearer YOUR_API_KEY`. Use this only for a dedicated integration: everyone using that connector then
acts as the integration user, with full rights and no role limits.

---

## Claude Code

Sign in with your CRM account (recommended for one person):

```bash
claude mcp add --transport http --scope user inflowcrm https://mcp.inflowcrm.pl
```

Then run `/mcp`, select `inflowcrm` and choose **Authenticate**. The browser opens the sign-in page. Without
`--scope user` the server is added for the current project only.

On a machine without a browser (for example over SSH), run `claude mcp login inflowcrm --no-browser`, open the printed
address on your own computer, and paste the callback address back when asked.

**With an API key,** keep the key out of the repository: put a reference in the project's `.mcp.json` and set
`INFLOWCRM_API_KEY` in your environment.

```json
{
  "mcpServers": {
    "inflowcrm": {
      "type": "http",
      "url": "https://mcp.inflowcrm.pl",
      "headers": { "Authorization": "Bearer ${INFLOWCRM_API_KEY}" }
    }
  }
}
```

**Check the connection** with `claude mcp list` (`Connected`, `Needs authentication` or `Failed to connect`), or with
`/mcp` inside Claude Code. Claude Code reads the tool list when it connects, so after InflowCRM enables more tools,
run `/mcp` and reconnect.

**Read-only assistant:** add this to `.claude/settings.json`. It blocks these tools in this Claude Code setup only;
it does not limit the key itself. Review the list when InflowCRM adds tools, because new tools are not blocked.

```json
{
  "permissions": {
    "deny": [
      "mcp__inflowcrm__create_record", "mcp__inflowcrm__create_records",
      "mcp__inflowcrm__update_record", "mcp__inflowcrm__update_records",
      "mcp__inflowcrm__delete_record", "mcp__inflowcrm__generate_document",
      "mcp__inflowcrm__create_sms_template", "mcp__inflowcrm__update_sms_template", "mcp__inflowcrm__delete_sms_template",
      "mcp__inflowcrm__create_email_template", "mcp__inflowcrm__update_email_template", "mcp__inflowcrm__delete_email_template",
      "mcp__inflowcrm__create_document_template", "mcp__inflowcrm__update_document_template", "mcp__inflowcrm__delete_document_template",
      "mcp__inflowcrm__create_field", "mcp__inflowcrm__update_field", "mcp__inflowcrm__delete_field",
      "mcp__inflowcrm__create_list_view", "mcp__inflowcrm__update_list_view", "mcp__inflowcrm__delete_list_view",
      "mcp__inflowcrm__create_role", "mcp__inflowcrm__update_role", "mcp__inflowcrm__delete_role"
    ]
  }
}
```

**Skill (optional):** `/plugin marketplace add InflowCRM/public-api-docs`, then `/plugin install inflowcrm@inflowcrm`
([details](skill.md)).

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
   as in steps 4–5 for [Claude](#claude-desktop-and-claudeai): your CRM address, log in, allow.
6. In a chat, type `@` and pick the plugin, then ask e.g. *"How many customers do we have?"*.

ChatGPT asks for confirmation before write actions. Review what the tool is about to send, and keep the "remember"
option off for tools that change or delete data. When InflowCRM enables more tools for your company, open the app's
details page in the app settings and refresh it; you can also switch individual tools off there.

Skill: paste the [skill text](skill.md#copy-and-paste) into the project's or custom GPT's instructions.

---

## Gemini

### Gemini app

> **Set Gemini to English.** Custom apps are visible only in English, so use
> [gemini.google.com/customize/connectors?hl=en](https://gemini.google.com/customize/connectors?hl=en). This works in
> Poland too, including on the free plan.

1. On a computer, open [the English connectors page](https://gemini.google.com/customize/connectors?hl=en) (custom apps
   can only be connected on the web; the mobile app can use them afterwards).
2. Under **Custom apps**, choose **Add a custom app**.
3. Enter `https://mcp.inflowcrm.pl` and click **Next**. Leave the advanced fields empty — Gemini registers itself.
4. On the page *"Połącz aplikację AI z InflowCRM"* enter your CRM address, log in and allow access. The consent
   screen warns that the address is not a known AI app; [this is expected](troubleshooting.md#the-consent-screen-warns-about-an-unknown-address)
   when the address belongs to Google and you started the connection yourself.
5. In a prompt, type `@` and pick the app.

Gemini asks for confirmation before write actions. Skill: paste the [skill text](skill.md#copy-and-paste) into your
prompt or a Gem's instructions.

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

### Gemini Enterprise

The custom MCP connector in Gemini Enterprise needs an OAuth client registered in advance, which is not covered by the
sign-in flow above. [Contact us](../contact.md) if your company wants to connect it.

---

## Perplexity

Not yet tested by InflowCRM. Steps follow
[Perplexity's guide](https://www.perplexity.ai/help-center/en/articles/13915507-adding-custom-remote-connectors).
Custom connectors need a paid plan (Pro, Max or Enterprise). In an organization, an administrator must turn on
**Allow members to add custom connectors** or add the connector for everyone.

1. Open **Plugins** in the sidebar (or **Customize → Connectors**), then **Add custom → Add MCP connector** and choose **Remote**.
2. Name it `InflowCRM` and enter the URL `https://mcp.inflowcrm.pl`.
3. Under **Advanced**, set **Authentication** to **OAuth** and **Transport** to **Streamable HTTP**. Leave the client
   credentials empty.
4. Tick the acknowledgement, click **Add**, then click the connector card to sign in: your CRM address, log in, allow.
   The consent screen [warns about an unknown address](troubleshooting.md#the-consent-screen-warns-about-an-unknown-address).

Perplexity's form has no field for custom request headers, so use signing in rather than an API key.

---

## Grok

Not yet tested by InflowCRM. Steps follow [xAI's guide](https://docs.x.ai/grok/connectors). On Business and Enterprise
plans a team admin may have to set up connectors first.

1. Go to [grok.com/connectors](https://grok.com/connectors), click **New Connector**, then **Custom**.
2. Enter the URL `https://mcp.inflowcrm.pl`.
3. Sign in when asked: your CRM address, log in, allow. The consent screen
   [warns about an unknown address](troubleshooting.md#the-consent-screen-warns-about-an-unknown-address).

---

## Limit what each person can do

**For now MCP is available to company administrators only**, so there is no separate access per employee yet.
We will soon add an option that lets employees use MCP within the permissions of their own CRM role.

Until then:

| You want | Use | Who enforces it |
|---|---|---|
| A prompt before every change | Approval settings in your AI app | Your AI app only |
| A read-only script or report | Tool filter in the client, e.g. the [Claude Code deny list](#claude-code) or **Tools to include** in n8n | That client only. **The key itself keeps full rights** |

- A company administrator and an **API key have full rights in your company's data**. Do not rely on a role to limit a key.
- To cut off access, block the user in the CRM. This also ends their CRM access.

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

## Not supported

- **SSE-only clients** — the server speaks Streamable HTTP only.
- **Browser-based clients** calling the server directly from a web page — browser `Origin` headers are rejected.
- **Signing in as an integration user** — integration users connect with their API key only.
