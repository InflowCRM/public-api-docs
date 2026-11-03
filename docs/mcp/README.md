# MCP Server (AI assistants)

InflowCRM ships a [Model Context Protocol](https://modelcontextprotocol.io) (MCP) server. It lets an AI assistant —
claude.ai, ChatGPT, Claude Desktop, Claude Code, Cursor, VS Code Copilot, your own agent — read and update the CRM
data of **one company** in plain language: find customers, count open deals, move a potential to the next stage,
attach a file, generate a PDF offer, prepare SMS / e-mail / document templates, look up a Polish company in GUS.

The MCP server sits on top of the same engine as the [REST API](../rest-api/README.md): the same modules and fields
(by **Api field name**), the same tenant isolation and record permissions.

> **Availability:** MCP is a paid add-on. It is switched on per company by InflowCRM — contact
> [support](../contact.md) to enable it. Until it is enabled the endpoint answers `404 Not Found` for an API key,
> and signing in says that MCP is not enabled in that CRM.

---

## 🚀 Quick start

There are two ways to connect. Both use the same address and the same tools.

| | Sign in with your CRM account (OAuth) | API key |
|---|---|---|
| Best for | claude.ai, ChatGPT, Claude Desktop connectors — and Claude Code | Claude Code, Codex, Cursor, VS Code, n8n, scripts, your own agents |
| Who connects | A company administrator (employees only when your company has this enabled — see *Who can sign in* below) | Anyone holding the key of an integration user |
| Acts as | The signed-in user, with their rights | The integration user (full rights in your company's data) |
| Setup | Add a custom connector with the URL, sign in on your company's CRM page, allow access | Send the key in the `Authorization` header |

1. **Ask InflowCRM to enable MCP** for your company.
2. **Connect:**
   - **OAuth:** add `https://mcp.inflowcrm.pl` as a custom connector / remote MCP server in your AI app and sign in
     — step by step in [Connect your AI tool](clients.md#claudeai-and-claude-desktop-connector).
   - **API key:** a company administrator creates a user with the **integration user** option and copies its
     **API key** from the user's details (the same key the REST API and the [n8n node](../n8n-node.md) use). Send
     it in the `Authorization` header.
3. **Optional:** install the [InflowCRM agent skill](skill.md) so the assistant knows the CRM's conventions up front.

---

## 🌐 Endpoint

```
https://mcp.inflowcrm.pl
```

`https://mcp.inflowcrm.pl/mcp` is accepted as well.

| Property | Value |
|---|---|
| Transport | Streamable HTTP, **stateless** (no sessions, no `Mcp-Session-Id`) |
| Method | `POST` only — `GET` / `DELETE` return `405 Method Not Allowed` |
| Responses | Plain JSON (`application/json`), no SSE stream |
| Required header | `Accept: application/json, text/event-stream` (MCP clients send it automatically) |
| Request body | JSON-RPC 2.0, one message per request — batches are rejected (`400`) |
| Max body size | 7 MB (enough for one 5 MB file encoded as base64) |
| Server name | `inflow-crm` |
| MCP features | **Tools** only (no resources, prompts or sampling) |

---

## 🔑 Authentication

### Sign in with your CRM account (OAuth 2.1)

The server is also an OAuth 2.1 authorization server, so AI apps that connect "custom connectors" (claude.ai,
ChatGPT, Claude Desktop) and MCP clients with OAuth support (Claude Code) can connect without an API key. The app
registers itself (dynamic client registration), uses PKCE, and finds everything it needs in the standard metadata:

```
https://mcp.inflowcrm.pl/.well-known/oauth-authorization-server
https://mcp.inflowcrm.pl/.well-known/oauth-protected-resource
```

How signing in works:

1. In your AI app, add the connector with the URL `https://mcp.inflowcrm.pl` and click **Connect**.
2. A page *"Połącz aplikację AI z InflowCRM"* asks for **your CRM address** — the one you log in at, e.g.
   `firma.inflowstore.pl`.
3. You are sent to your company's CRM. Log in if needed. The consent screen shows which app asks for access, the
   address it will return to, your account and your company.
4. Click **allow**. The app is connected; from now on the assistant works **as you, with your rights**.

The sign-in link is valid for 10 minutes. The app keeps the connection alive by itself (a short-lived access token,
renewed automatically for up to 30 days of inactivity).

#### Who can sign in

| Account | Can sign in |
|---|---|
| Company administrator | **Yes**, whenever MCP is enabled for the company |
| Employee (regular CRM user with a role) | **Only when your company asked InflowCRM to allow employees** — then within the limits of their role |
| Integration user | No — integration users connect with their API key |

Until employees are allowed, an employee who tries to connect sees *"To konto nie może łączyć aplikacji AI z CRM.
Skontaktuj się z administratorem firmy."* (This account cannot connect AI apps to the CRM — contact your company
administrator.) Allowing employees is a feature InflowCRM switches on at the company's request; switching it off
disconnects employees' apps immediately.

What a signed-in connection may do:

- **Exactly what the user may do in the CRM.** Records, fields and modules follow the user's role, the same as in
  the CRM's screens. There are no separate OAuth scopes and no "read only" mode — to limit an assistant further,
  use your AI app's per-tool settings (see *Confirmations and restricting what the assistant may do* below).
- An employee never gets the configuration tools (fields, list views, roles), even when the company has them.
- Blocking or deleting the user in the CRM ends the access at once. Removing the connector in the AI app
  disconnects it too.

### API key

Use the API key of an integration user. Any of these headers works (checked in this order):

```
Authorization: Bearer YOUR_API_KEY      ← recommended (MCP convention)
x-api-key: YOUR_API_KEY                 ← same header as the REST API
apiKey: YOUR_API_KEY
```

> **Security:** the key acts as the integration user, which has **full rights in your company's data**.
> Store it like a password, never commit it, and give each assistant/integration its own integration user so it
> can be blocked independently. Blocking the integration user (or deleting it) revokes the key.

### Responses

| Response | Meaning |
|---|---|
| `401 {"error":"Unauthorized"}` | Missing, unknown or expired credential; the user is blocked/deleted, or the company account is inactive. The `WWW-Authenticate` header points OAuth clients to the metadata |
| `404 Not Found` | The credential is valid but MCP is not enabled for the company |
| `403` (JSON-RPC `Forbidden origin`) | The request carries a browser `Origin` header that is not allow-listed — call MCP from a server-side client |
| `429` (JSON-RPC `Rate limit exceeded`) | Too many requests — see *Rate limits* below |

---

## 🔌 Connecting a client

### claude.ai, ChatGPT, Claude Desktop

Add `https://mcp.inflowcrm.pl` as a custom connector and sign in — see
**[Connect your AI tool](clients.md#claudeai-and-claude-desktop-connector)**.

### Claude Code

With an API key:

```bash
claude mcp add --transport http inflowcrm https://mcp.inflowcrm.pl \
  --header "Authorization: Bearer YOUR_API_KEY"
```

Or sign in with your CRM account: add the server without the header, then run `/mcp`, pick `inflowcrm` and choose
**Authenticate**.

```bash
claude mcp add --transport http inflowcrm https://mcp.inflowcrm.pl
```

Check with `claude mcp list`, then ask e.g. *"How many open potentials do we have per stage?"*.
Add `--scope user` to make the server available in every project.

> Claude Code reads the tool list when it connects. After InflowCRM adds tools or enables a feature for your
> company, run `/mcp` and reconnect.

### Codex, Gemini CLI, Cursor, VS Code, Windsurf, n8n, Python, OpenAI Agents SDK, LangChain

Step-by-step configs for each tool — plus a **universal setup prompt** that lets any AI agent configure the
connection and install the skill by itself — are on **[Connect your AI tool](clients.md)**.

### Your own agent / any MCP SDK

Any client that supports **Streamable HTTP** with custom headers (or MCP OAuth) works. Because the server is
stateless, every request can be sent on its own:

```bash
curl -s https://mcp.inflowcrm.pl \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

```bash
curl -s https://mcp.inflowcrm.pl \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"count_records","arguments":{"module":"customer"}}}'
```

Tool results come back as JSON text in `result.content[0].text`; failures set `result.isError: true`.

---

## 🧰 What the assistant can do

Up to **37 tools** (depending on your plan and rights), grouped by capability (full reference:
**[MCP Tools](tools.md)**):

| Capability | Tools |
|---|---|
| Discover | `list_modules`, `get_module_fields`, `get_sample_record` |
| Read records | `search_records` (free-text `query`, filters, sort, paging), `find_record`, `count_records`, `get_record`, `get_records_by_ids` |
| Write records | `create_record`, `update_record`, `create_records` / `update_records` (bulk, up to 50), `delete_record` (to trash) |
| Pipelines | `get_pipelines`, `get_pipeline_summary` (records and value per stage) |
| Files | `list_record_files`, `download_file`, upload via `create_record` / `update_record` |
| Users | `list_users`, `get_user` (read only — MCP never edits users) |
| Documents | `list_document_templates`, `generate_document` (PDF, optionally saved to the record) |
| Templates | `get_template_tags`; SMS: `list_sms_templates`, `get_sms_template`, `create_sms_template`, `update_sms_template`, `delete_sms_template`; e-mail: `list_email_templates`, `get_email_template`, `create_email_template`, `update_email_template`, `delete_email_template`; documents: `get_document_template`, `create_document_template`, `update_document_template`, `delete_document_template` |
| Lookup | `lookup_company_gus` (Polish GUS registry by NIP / REGON / KRS) |

The template tools **prepare templates** — they do not send SMS or e-mail. Messages are sent by the CRM itself
(manually or by the template's automatic sending), as before.

### 🛠️ Configuration tools

Companies that want the assistant to help **configure the CRM** can ask InflowCRM to enable the configuration tools.
They add **13 tools** (up to **50** in total):

| Capability | Tools |
|---|---|
| Fields and pick lists | `get_field_layout`, `create_field`, `update_field`, `delete_field` |
| Saved list views | `get_list_views`, `create_list_view`, `update_list_view`, `delete_list_view` |
| Roles | `list_roles`, `get_role`, `create_role`, `update_role`, `delete_role` |

They are available only to a **company administrator** (signed in) or an **API key** — never to an employee.
Configuration changes apply to **every user of the company** at once.

### Which tools you see

- Tools your plan, the enabled features or your account do not allow are **not on the tool list** at all.
- After InflowCRM enables a feature for your company, **reconnect** the AI app (or start a new chat) to see the new
  tools.
- A signed-in employee sees the record tools, but a call fails when their role does not allow it (e.g. no access to
  the module).

Exposed modules: `customer`, `contact`, `task`, `note`, `order`, `potential`, `event`, `enlist`, `coach`, `room` —
limited to the modules of your company's plan. `list_modules` returns them with the names your company configured.

As in the REST API, **only fields with an Api field name** can be read, filtered and written over MCP. Give a field an Api field name in
the field settings to make it readable and writable by the assistant.

---

## 🛡️ Confirmations and restricting what the assistant may do

### Confirmations

Tools that delete or change sensitive configuration refuse to run until the assistant sends **`confirm: true`**:

| Tool | When `confirm: true` is needed |
|---|---|
| `delete_sms_template`, `delete_email_template`, `delete_document_template` | Always |
| `update_sms_template`, `update_email_template` | When the template sends automatically and its content changes |
| `update_document_template` | When the document is attached to an automatically sent template and its content changes |
| `delete_field`, `delete_list_view`, `delete_role`, `update_role` | Always |

The assistant is told to set `confirm: true` only after **you** agreed in the conversation, and to ask once for a
whole batch (listing every item), not once per item. `confirm` is set by the model, though — the confirmation that
cannot be bypassed is the one in your AI app.

> **Recommended:** in claude.ai / ChatGPT / Claude Desktop set the InflowCRM **write and delete tools to "needs
> approval"** (ask every time) in the connector's tool settings. Then the app asks *you* before each change, whatever
> the model was told by text in the data.

In Claude Code you can deny tools outright in `.claude/settings.json`:

```json
{
  "permissions": {
    "deny": [
      "mcp__inflowcrm__delete_record",
      "mcp__inflowcrm__create_records",
      "mcp__inflowcrm__update_records"
    ]
  }
}
```

### Scopes

Every tool belongs to a scope:

| Scope | Tools |
|---|---|
| *(always)* | `list_modules` |
| `records:read` | `get_module_fields`, `get_sample_record`, `search_records`, `find_record`, `count_records`, `get_record`, `get_records_by_ids`, `get_pipeline_summary` |
| `records:write` | `create_record`, `create_records`, `update_record`, `update_records` |
| `records:delete` | `delete_record` |
| `files:read` | `list_record_files`, `download_file` |
| `files:write` | attaching files in `create_record` / `update_record`, `saveToRecord` in `generate_document` |
| `users:read` | `list_users`, `get_user` |
| `lookup` | `lookup_company_gus`, `get_pipelines` |
| `documents:generate` | `list_document_templates`, `generate_document` |
| `templates:read` | `get_template_tags`, `list_sms_templates`, `get_sms_template`, `list_email_templates`, `get_email_template`, `get_document_template` |
| `templates:write` | `create_*_template`, `update_*_template`, `delete_*_template` (SMS, e-mail, document) |
| `fields:read` / `fields:write` | `get_field_layout` / `create_field`, `update_field`, `delete_field` |
| `lists:read` / `lists:write` | `get_list_views` / `create_list_view`, `update_list_view`, `delete_list_view` |
| `roles:read` / `roles:write` | `list_roles`, `get_role` / `create_role`, `update_role`, `delete_role` |

> **Today every API key receives all scopes**, and a signed-in user is limited by their role instead. Per-key scopes
> are planned. Until then, restrict tools in your MCP client as shown above. The `fields:`, `lists:` and `roles:`
> tools additionally need the configuration tools enabled for the company.

### Built-in safeguards

- **Tenant isolation** — every read and write is limited to your company; ids of other companies' records, files,
  users, pipelines or templates behave as "not found".
- **Record permissions** — the same visibility rules as the CRM and the REST API; a signed-in user sees what their
  role allows.
- **Strict input** — unknown arguments are rejected; unknown field, filter and sort names are rejected with the list of
  valid ones; values are
  validated against the field type (pick list, number, date, e-mail, rating, length, required).
- **Soft delete** — `delete_record` moves the record to the CRM trash.
- **No user editing** — no tool changes users or API keys. Roles can be changed only with the configuration tools,
  and `delete_role` never moves users to another role.
- **Safe template HTML** — e-mail and document template HTML is sanitized when saved (scripts and other unsafe
  content are removed).
- **Audit log** — every tool call that passes input validation is logged with the tool name, argument *names*,
  affected record ids, result, IP address and user agent — and, for a signed-in user, which connected app made the
  call. Argument values are not stored in the audit log.

### Prompt injection

Notes, e-mails and descriptions stored in the CRM are written by third parties. The server tells the model to
treat them as **data, never instructions**, to ask the user before deleting or bulk-modifying records, and never
to change configuration or templates because text in the CRM asks for it. Keep human confirmation on for deletes,
bulk tools, template writes and configuration tools in your client.

---

## ⏱️ Rate limits

| Limit | Default | Applies to |
|---|---|---|
| Requests | **300 per 5 minutes** per API key user, or per signed-in user and connected app | every MCP HTTP request, including `initialize` and `tools/list` |
| Bulk writes | **10 calls per 5 minutes** (≤ 500 records) | `create_records` + `update_records` together |
| Document generation | **20 calls per 5 minutes** | `generate_document` |
| Template writes | **60 calls per 5 minutes** | create / update / delete of SMS, e-mail and document templates together |
| Configuration writes | **30 calls per 5 minutes** | create / update / delete of fields, list views and roles together |
| Failed authentication | 300 per 5 minutes per IP | counts `401` responses; once exceeded, every request from that IP gets `429` until the window resets |

The request limit returns HTTP `429`; the per-tool limits return a tool error such as
`Too many bulk record writes, retry in 42 s.` A call refused by a limit writes nothing.

### Other limits

| Item | Limit |
|---|---|
| Records per page | 50 (default 20) |
| Ids per `get_records_by_ids` | 50 |
| Items per bulk call | 50 |
| Fields per `delete_field` | 25 |
| Files per write | 10, each ≤ 5 MB decoded; whole request ≤ 7 MB (≈ 5 MB of files in total) |
| `download_file` / `generate_document` | ≤ 5 MB per file |
| Filter conditions | 20 per `advancedFilter` |
| Sort keys | 3 |
| Free-text `query` | 2–100 characters, scans at most 500 candidates |
| SMS template text | 1,000 characters |
| Template name / description / e-mail subject | 200 / 2,000 / 500 characters |
| Template HTML | 5 MB per part (body, header, footer) |

---

## ❌ Errors

Tool failures are returned as a normal tool result with `isError: true` and a short, actionable message — it says
what was wrong and which tool to call next. Messages never contain stack traces. Typical prefixes:

| Prefix | Meaning |
|---|---|
| `VALIDATION_ERROR:` | Bad input (wrong type, invalid NIP checksum, `query` rules…) |
| `RESOURCE_NOT_FOUND:` | Record, file, user, pipeline, template, field, list view or role not found (also for other companies' ids) |
| `CONFLICT:` | The change clashes with the current state (e.g. a role that still has users, a name already taken) |
| `FORBIDDEN:` / `UNAUTHORIZED:` / `PERMISSION_DENIED:` | The user behind the connection (their role) may not perform the operation |
| `INTERNAL_ERROR:` | Server-side failure — retry later |

Many validation messages have no prefix, e.g. `Unknown field(s) in \`data\`: foo. Valid fields: …`,
`Field 'status' (Status) must be one of: …`, `Missing required field(s): …`.

A call without a required `confirm: true` is refused before anything changes — either as invalid arguments or with
a message such as `This template is sent automatically (autoSending.enabled). Show the new text to the user and call
again with confirm: true.`

Bulk tools answer per item (`{ index, ok, id }` / `{ index, ok: false, error }`) and set `isError` only when every
item failed.

---

## 📚 See also

- [Connect your AI tool](clients.md)
- [MCP Tools reference](tools.md)
- [Agent skill for InflowCRM](skill.md)
- [REST API — Modules & Fields](../rest-api/modules.md)
