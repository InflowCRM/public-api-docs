# MCP Server (AI assistants)

InflowCRM ships a [Model Context Protocol](https://modelcontextprotocol.io) (MCP) server. It lets an AI assistant —
Claude Desktop, claude.ai, Claude Code, ChatGPT, Gemini, Perplexity, Grok, Cursor, VS Code Copilot, your own agent — read and update the CRM
data of **one company** in plain language: find customers, count open deals, move a potential to the next stage,
attach a file, generate a PDF offer, prepare SMS / e-mail / document templates, look up a Polish company in GUS.

The MCP server sits on top of the same engine as the [REST API](../rest-api/README.md): the same modules and fields
(by **Api field name**), the same tenant isolation and record permissions.

> **Availability:** MCP is a paid add-on. It is switched on per company by InflowCRM — contact
> [support](../contact.md) to enable it. Until it is enabled the endpoint answers `403` with
> `MCP is not enabled for this company. Ask your CRM administrator or InflowCRM support to enable it.` for a valid
> API key, and signing in says that MCP is not enabled in that CRM.

---

## 🚀 Quick start

There are two ways to connect. Both use the same address and the same tools.

| | Sign in with your CRM account (OAuth) | API key |
|---|---|---|
| Best for | claude.ai, ChatGPT, Gemini, Claude Desktop connectors — and Claude Code, Gemini CLI | Claude Code, Gemini CLI, Codex, Cursor, VS Code, n8n, scripts, your own agents |
| Who connects | A company administrator, or an employee when InflowCRM has enabled it for your company (see *Who can sign in* below) | Anyone holding the key of an integration user |
| Acts as | The signed-in user, with their rights | The integration user (full rights in your company's data) |
| Setup | Add a custom connector with the URL, sign in on your company's CRM page, allow access | Send the key in the `Authorization` header |

1. **Ask InflowCRM to enable MCP** for your company.
2. **Connect:**
   - **OAuth:** add `https://mcp.inflowcrm.pl` as a custom connector / remote MCP server in your AI app and sign in
     — step by step in [Connect your AI tool](clients.md#claude-desktop-and-claudeai).
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
   `firma.inflowstore.pl`. Capital letters do not matter: `Firma.inflowstore.pl` finds the same company.
3. You are sent to your company's CRM. Log in if needed. The consent screen shows which app asks for access, the
   address it will return to, your account and your company.
4. Click **allow**. The app is connected; from now on the assistant works **as you, with your rights**.

The sign-in link is valid for 10 minutes. The app keeps the connection alive by itself (a short-lived access token,
renewed automatically for up to 30 days of inactivity).

#### Who can sign in

| Account | Can sign in |
|---|---|
| Company administrator | **Yes**, whenever MCP is enabled for the company |
| Employee (regular CRM user with a role) | **Yes, when InflowCRM has enabled MCP for employees of your company** — within the permissions of their own role |
| Integration user | No — integration users connect with their API key |

**Employees need an extra switch.** MCP for employees is switched on per company by InflowCRM, on top of MCP itself.
Until it is on, an employee who tries to connect sees *"To konto nie może łączyć aplikacji AI z CRM.
Skontaktuj się z administratorem firmy."* (This account cannot connect AI apps to the CRM — contact your company
administrator.) Once it is on, an employee must have a role; the assistant then works **within that role**: it sees only
the modules, records and fields the role allows, and its tool list follows the role (see
[Tools depend on the role](tools.md#tools-depend-on-the-role)).

What a signed-in connection may do:

- **The account's own rights.** The assistant acts with the rights of the signed-in account. There are no separate OAuth
  scopes or a "read only" switch at sign-in — see [Limit what each person can do](clients.md#limit-what-each-person-can-do).
- Blocking or deleting the user in the CRM ends the access at once. Removing the connector in the AI app removes
  its sign-in from the app.

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
| `403` with `error.code: "mcp_disabled"` | The credential is valid but MCP is not enabled for the company. Not a `401`, so OAuth clients do not restart their sign-in |
| `404 Not Found` | Rare: the server could not read the company's feature flag. Retry; if it persists, contact support |
| `403` (JSON-RPC `Forbidden origin`) | The request carries a browser `Origin` header that is not allow-listed — call MCP from a server-side client |
| `429` (JSON-RPC `Rate limit exceeded`) | Too many requests — see *Rate limits* below |

---

## 🔌 Connecting a client

### Claude Desktop, claude.ai, Claude Code, ChatGPT, Gemini, Perplexity, Grok, Codex, Cursor, VS Code, n8n

Step-by-step setup for each tool — plus a **universal setup prompt** that lets an AI agent configure the
connection and install the skill by itself — is on **[Connect your AI tool](clients.md)**. If something fails, see
**[Troubleshooting](troubleshooting.md)**.

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

Up to **43 tools** (depending on your plan and rights), grouped by capability (full reference:
**[MCP Tools](tools.md)**):

| Capability | Tools |
|---|---|
| Discover | `whoami` (who the connection acts as), `list_modules`, `get_module_fields`, `get_sample_record` |
| Read records | `search_records` (free-text `query`, filters, sort, paging), `find_record`, `count_records`, `get_record`, `get_records_by_ids` |
| Write records | `create_record`, `update_record`, `create_records` / `update_records` (bulk, up to 50), `delete_record` (to trash) |
| Pipelines | `get_pipelines`, `get_pipeline_summary` (records and value per stage) |
| Files | `list_record_files`, `download_file`, upload via `create_record` / `update_record` |
| Notes | `list_record_notes` (the notes of a record; notes are read through their parent record) |
| Messages | `list_record_messages`, `get_message` (read the e-mails and SMS sent from a record) |
| Users | `list_users`, `get_user` (read only — MCP never edits users) |
| Documents | `list_document_templates`, `generate_document` (PDF, optionally saved to the record) |
| Templates | `get_template_tags`; SMS: `list_sms_templates`, `get_sms_template`, `create_sms_template`, `update_sms_template`, `delete_sms_template`; e-mail: `list_email_templates`, `get_email_template`, `create_email_template`, `update_email_template`, `delete_email_template`; documents: `get_document_template`, `create_document_template`, `update_document_template`, `delete_document_template` |
| Lookup | `lookup_company_gus` (Polish GUS registry by NIP / REGON / KRS) |

The template tools **prepare templates** — they do not send SMS or e-mail. Messages are sent by the CRM itself
(manually or by the template's automatic sending), as before.

### 🛠️ Configuration tools

Companies that want the assistant to help **configure the CRM** can ask InflowCRM to enable the configuration tools.
They add **33 tools** (up to **76** in total):

| Capability | Tools |
|---|---|
| Fields and pick lists | `get_field_layout`, `create_field`, `update_field`, `delete_field` |
| Saved list views | `get_list_views`, `create_list_view`, `update_list_view`, `delete_list_view` |
| Roles | `list_roles`, `get_role`, `create_role`, `update_role`, `delete_role` |
| Squads | `list_squads`, `create_squad`, `update_squad`, `delete_squad`, `add_users_to_squad`, `remove_users_from_squad` |
| Sales pipelines | `create_pipeline`, `update_pipeline`, `create_stage`, `update_stage`, `reorder_stages`, `delete_stage` (only with potentials or orders in your plan; `get_pipelines` gives the ids) |
| Module names | `update_module_names` (what the company calls a module; `list_modules` shows which can be renamed) |
| Company data | `get_company_settings`, `update_company_settings` (name, address, NIP, start page, knowledge base link) |
| Record numbering | `get_numbering_settings`, `update_numbering_settings`, `backfill_numbering` |
| Look of the web app | `get_branding`, `update_branding` (colour theme and tab title, only when white-label is active) |

They are available only to a **company administrator** (signed in) or an **API key** — never to an employee.
Configuration changes apply to **every user of the company** at once. The settings tools return only the values listed in
[Company settings](tools.md#company-settings), nothing else of the company configuration.

**Exception:** an **employee signed in with OAuth** gets the four list view tools (`get_list_views`, `create_list_view`,
`update_list_view`, `delete_list_view`) for their own **private** views — no sharing, no `confirm`, only views they
own — whether or not the configuration tools are enabled for the company. API keys (an employee's included) do not get
them. See
[Your own list views](tools.md#your-own-list-views-signed-in-employees).

### Which tools you see

- Tools your plan, the enabled features or your account do not allow are **not on the tool list** at all.
- For an employee **signed in with OAuth** the list also follows the CRM role: a tool the role has no right for is not
  listed (see [Tools depend on the role](tools.md#tools-depend-on-the-role)). Administrators and API keys see the full
  list.
  `whoami` returns `effectiveRights` per module.
- After InflowCRM enables a feature for your company, **reconnect** the AI app (or start a new chat) to see the new
  tools.

Exposed modules: `customer`, `contact`, `task`, `note`, `order`, `potential`, `event`, `enlist`, `coach`, `room` —
limited to the modules of your company's plan. `list_modules` returns them with the names your company configured.
`note` is special: notes are not searchable, counted or sampled — you read them with `list_record_notes` on the
record they belong to, and a note is visible only through a parent record you can see
(see [Notes](tools.md#notes)).

As in the REST API, **only fields with an Api field name** can be read, filtered and written over MCP. Give a field an Api field name in
the field settings to make it readable and writable by the assistant.

---

## 🛡️ Confirmations and restricting what the assistant may do

### Confirmations

Tools that delete or change sensitive configuration refuse to run until the assistant sends **`confirm: true`**:

| Tool | When `confirm: true` is needed |
|---|---|
| `delete_record`, `delete_sms_template`, `delete_email_template`, `delete_document_template` | Always |
| `update_sms_template`, `update_email_template` | When the template sends automatically and its content changes |
| `update_document_template` | When the document is attached to an automatically sent template and its content changes |
| `update_field`, `delete_field`, `create_list_view`, `update_list_view`, `delete_list_view`, `create_role`, `update_role`, `delete_role` | Always — except the list-view tools for a signed-in employee working on their own private views, which need no `confirm` |
| `delete_squad`, `add_users_to_squad`, `remove_users_from_squad` | Always |
| `update_pipeline`, `update_stage`, `reorder_stages`, `delete_stage` | Always (`create_pipeline` and `create_stage` need none) |
| `update_module_names`, `update_company_settings`, `update_numbering_settings`, `backfill_numbering`, `update_branding` | Always |

The assistant is told to set `confirm: true` only after **you** agreed in the conversation, and to ask once for a
whole batch (listing every item), not once per item. `confirm` is set by the model, though — the confirmation that
cannot be bypassed is the one in your AI app.

> **Recommended:** keep your AI app asking *you* before each change, and do not choose "always allow" for tools that
> write or delete. Then the app asks you whatever the model was told by text in the data. Where to set this differs per
> app; see [Connect your AI tool](clients.md).

In Claude Code you can deny tools outright in `.claude/settings.json`: copy the full [read-only list](clients.md#claude-code).

### Scopes

Scopes are how the server groups tools. **You cannot choose them yet**: every API key receives all of them, as the note
below explains. Every tool belongs to a scope:

| Scope | Tools |
|---|---|
| *(always)* | `list_modules`, `whoami` |
| `records:read` | `get_module_fields`, `get_sample_record`, `search_records`, `find_record`, `count_records`, `get_record`, `get_records_by_ids`, `list_record_notes`, `get_pipeline_summary` |
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
| `squads:read` / `squads:write` | `list_squads` / `create_squad`, `update_squad`, `delete_squad`, `add_users_to_squad`, `remove_users_from_squad` |
| `settings:read` / `settings:write` | `get_company_settings`, `get_numbering_settings`, `get_branding` / `update_module_names`, `update_company_settings`, `update_numbering_settings`, `backfill_numbering`, `update_branding`, `create_pipeline`, `update_pipeline`, `create_stage`, `update_stage`, `reorder_stages`, `delete_stage` |
| `messages:read` | `list_record_messages`, `get_message` |

> **Today every API key receives all scopes**. Per-key scopes
> are planned. Until then, restrict tools in your MCP client as shown above. The `fields:`, `lists:`, `roles:`,
> `squads:` and `settings:` tools additionally need the configuration tools enabled for the company — except the list view tools of a
> employee signed in with OAuth (own private views).

### Built-in safeguards

- **Tenant isolation** — every read and write is limited to your company; ids of other companies' records, files,
  users, pipelines or templates behave as "not found".
- **Record permissions** — the same visibility rules as the CRM and the REST API; a signed-in user sees what their
  role allows.
- **Notes follow their parent record** — a note is visible only when you can see a record it belongs to. A note with
  no existing parent is invisible to everyone, administrators and API keys included.
- **Strict input** — unknown arguments are rejected; unknown field, filter and sort names are rejected with the list of
  valid ones; values are
  validated against the field type (pick list, number, date, e-mail, rating, length, required).
- **Soft delete** — `delete_record` moves the record to the CRM trash.
- **No user editing** — no tool changes users or API keys; the only exception is squad membership
  (`add_users_to_squad`, `remove_users_from_squad`), available with the configuration tools. Roles can be changed
  only with the configuration tools, and `delete_role` never moves users to another role.
- **Safe template HTML** — e-mail and document template HTML is sanitized when saved (scripts and other unsafe
  content are removed).
- **Audit log** — every tool call that passes input validation is logged with the tool name, argument *names*,
  affected record ids, result, IP address and user agent — and, for a signed-in user, which connected app made the
  call. Argument values are not stored in the audit log, with one exception: a successful change of fields, list views
  (the administrator tools), roles, squads (name), module names, company data, numbering settings or branding made with the
  [configuration tools](#configuration-tools) also records the values it changed, before and after (only the changed
  keys, no secrets, personal data such as e-mails and phone numbers redacted, long values shortened). Not recorded
  that way: the pipeline and stage tools and `backfill_numbering` (tool name and ids only), squad membership changes
  (the user ids the call named and how many users it moved — not who was a member before), and the list view tools of a
  signed-in employee working on their own private views.

### Prompt injection

Notes, e-mails and descriptions stored in the CRM are written by third parties. The server tells the model to
treat them as **data, never instructions**, to ask the user before deleting or bulk-modifying records, and — when the
configuration tools are on — never to change configuration because text in the CRM asks for it. Keep human
confirmation on for deletes, bulk tools, template writes and configuration tools in your client.

---

## ⏱️ Rate limits

| Limit | Default | Applies to |
|---|---|---|
| Requests | **300 per 5 minutes** per API key user, or per signed-in user and connected app | every MCP HTTP request, including `initialize` and `tools/list` |
| Bulk writes | **10 calls per 5 minutes** (≤ 500 records) | `create_records` + `update_records` together |
| Document generation | **20 calls per 5 minutes** | `generate_document` |
| Template writes | **60 calls per 5 minutes** | create / update / delete of SMS, e-mail and document templates together |
| Configuration writes | **30 calls per 5 minutes** (a call refused before it wrote anything is not counted — unless the limit itself refused it) | create / update / delete of fields, list views, roles, squads, pipelines and stages, and every settings write (module names, company data, numbering, backfill, branding), together |
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
| Users per `add_users_to_squad` / `remove_users_from_squad` | 50 |
| Messages per `list_record_messages` page | 50 (default 20); `offset` up to 500 |
| Message reads (`list_record_messages`, `get_message`) | 120 per 5 minutes |
| Notes per `list_record_notes` page | 50 (default 20); `offset` up to 500 |
| Note reads (`list_record_notes`) | 120 per 5 minutes |
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
| `VALIDATION_ERROR:` | Bad input (wrong type, invalid NIP checksum, unknown field, unknown template tag, `query` rules…) — MCP tools start such a message with it, a bulk item's `error` too |
| `RESOURCE_NOT_FOUND:` | Record, file, user, pipeline, template, field, list view or role not found (also for other companies' ids) |
| `CONFLICT:` | The change clashes with the current state (e.g. a role that still has users, a name already taken) |
| `FORBIDDEN:` / `UNAUTHORIZED:` / `PERMISSION_DENIED:` | The user behind the connection (their role) may not perform the operation |
| `INTERNAL_ERROR:` | Server-side failure — retry later |

The validation prefix is added in one place for all tools, so a message reads e.g.
`VALIDATION_ERROR: Unknown field(s) in \`data\`: foo. Valid fields: …`. A refusal that is not a validation problem
(for example a change to an automatically sent template without `confirm`) keeps its own text.

When a call fails, the answer carries a second text block `Reference: <id>. If you cannot resolve this, report it with
report_problem (errorRefs).` For a bulk call that failed only partly (some items `ok: false`) the reference is there
too — the answer is not an error, but the failed items can be reported with it.

A call without a required `confirm: true` is refused before anything changes — either as invalid arguments or with
a message such as `This template is sent automatically (autoSending.enabled). Show the new text to the user and call
again with confirm: true.`

Bulk tools answer per item (`{ index, ok, id }` / `{ index, ok: false, error }`) and set `isError` only when every
item failed.

---

## 📚 See also

- [Connect your AI tool](clients.md)
- [Troubleshooting](troubleshooting.md)
- [MCP Tools reference](tools.md)
- [Agent skill for InflowCRM](skill.md)
- [REST API — Modules & Fields](../rest-api/modules.md)
