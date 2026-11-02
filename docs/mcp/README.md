# MCP Server (AI assistants)

InflowCRM ships a [Model Context Protocol](https://modelcontextprotocol.io) (MCP) server. It lets an AI assistant —
Claude Code, Cursor, VS Code Copilot, your own agent — read and update the CRM data of **one company** in plain
language: find customers, count open deals, move a potential to the next stage, attach a file, generate a PDF
offer, look up a Polish company in GUS.

The MCP server sits on top of the same engine as the [REST API](../rest-api/README.md): the same API key, the same
modules and fields (by **Api field name**), the same tenant isolation and record permissions.

> **Availability:** MCP is a paid add-on. It is switched on per company by InflowCRM — contact
> [support](../contact.md) to enable it. Until it is enabled the endpoint answers `404 Not Found` for your key.

---

## 🚀 Quick start

1. **Ask InflowCRM to enable MCP** for your company.
2. **Get an API key** — a company administrator creates a user with the **integration user** option and copies
   its **API key** from the user's details (the same key the REST API and the [n8n node](../n8n-node.md) use).
3. **Connect your MCP client** to the endpoint below, sending the key in the `Authorization` header.
4. **Optional:** install the [InflowCRM agent skill](skill.md) so the assistant knows the CRM's conventions up front.

---

## 🌐 Endpoint

```
https://srv.inflowcrm.pl/mcp
```

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

Use the API key of an integration user. Any of these headers works (checked in this order):

```
Authorization: Bearer YOUR_API_KEY      ← recommended (MCP convention)
x-api-key: YOUR_API_KEY                 ← same header as the REST API
apiKey: YOUR_API_KEY
```

| Response | Meaning |
|---|---|
| `401 {"error":"Unauthorized"}` | Missing or unknown key, or the integration user is blocked/deleted |
| `404 Not Found` | The key is valid but MCP is not enabled for the company |
| `403` (JSON-RPC `Forbidden origin`) | The request carries a browser `Origin` header that is not allow-listed — call MCP from a server-side client |
| `429` (JSON-RPC `Rate limit exceeded`) | Too many requests — see *Rate limits* below |

> **Security:** the key acts as the integration user, which has **full rights in your company's data**.
> Store it like a password, never commit it, and give each assistant/integration its own integration user so it
> can be blocked independently. Blocking the integration user (or deleting it) revokes the key.

---

## 🔌 Connecting a client

### Claude Code

```bash
claude mcp add --transport http inflowcrm https://srv.inflowcrm.pl/mcp \
  --header "Authorization: Bearer YOUR_API_KEY"
```

Check with `claude mcp list`, then ask e.g. *"How many open potentials do we have per stage?"*.
Add `--scope user` to make the server available in every project.

> Claude Code reads the tool list when it connects. After InflowCRM adds tools, run `/mcp` and reconnect.

### Cursor (config example)

`~/.cursor/mcp.json` or `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "inflowcrm": {
      "url": "https://srv.inflowcrm.pl/mcp",
      "headers": { "Authorization": "Bearer YOUR_API_KEY" }
    }
  }
}
```

### VS Code (config example)

`.vscode/mcp.json`:

```json
{
  "inputs": [
    { "type": "promptString", "id": "inflowcrm-key", "description": "InflowCRM API key", "password": true }
  ],
  "servers": {
    "inflowcrm": {
      "type": "http",
      "url": "https://srv.inflowcrm.pl/mcp",
      "headers": { "Authorization": "Bearer ${input:inflowcrm-key}" }
    }
  }
}
```

### Your own agent / any MCP SDK

Any client that supports **Streamable HTTP** with custom headers works. Because the server is stateless, every
request can be sent on its own:

```bash
curl -s https://srv.inflowcrm.pl/mcp \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

```bash
curl -s https://srv.inflowcrm.pl/mcp \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"count_records","arguments":{"module":"customer"}}}'
```

Tool results come back as JSON text in `result.content[0].text`; failures set `result.isError: true`.

> **claude.ai / Claude Desktop "custom connectors"** require OAuth, which the InflowCRM MCP server does not offer
> yet — they cannot send a static API key. Use Claude Code or a client that supports custom headers.

---

## 🧰 What the assistant can do

22 tools, grouped by capability (full reference: **[MCP Tools](tools.md)**):

| Capability | Tools |
|---|---|
| Discover | `list_modules`, `get_module_fields`, `get_sample_record` |
| Read records | `search_records` (free-text `query`, filters, sort, paging), `find_record`, `count_records`, `get_record`, `get_records_by_ids` |
| Write records | `create_record`, `update_record`, `create_records` / `update_records` (bulk, up to 50), `delete_record` (to trash) |
| Pipelines | `get_pipelines`, `get_pipeline_summary` (records and value per stage) |
| Files | `list_record_files`, `download_file`, upload via `create_record` / `update_record` |
| Users | `list_users`, `get_user` (read only — MCP never edits users) |
| Documents | `list_document_templates`, `generate_document` (PDF, optionally saved to the record) |
| Lookup | `lookup_company_gus` (Polish GUS registry by NIP / REGON / KRS) |

Exposed modules: `customer`, `contact`, `task`, `note`, `order`, `potential`, `event`, `enlist`, `coach`, `room` —
limited to the modules of your company's plan. `list_modules` returns them with the names your company configured.

As in the REST API, **only fields with an Api field name** are visible to MCP. Give a field an Api field name in
the field settings to make it readable and writable by the assistant.

---

## 🛡️ Scopes and restricting what the assistant may do

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

> **Today every API key receives all scopes.** Per-key scopes are planned. Until then, restrict tools in your MCP
> client. In Claude Code, for example, deny deletes and bulk writes in `.claude/settings.json`:
>
> ```json
> {
>   "permissions": {
>     "deny": [
>       "mcp__inflowcrm__delete_record",
>       "mcp__inflowcrm__create_records",
>       "mcp__inflowcrm__update_records"
>     ]
>   }
> }
> ```

Built-in safeguards regardless of scopes:

- **Tenant isolation** — every read and write is limited to your company; ids of other companies' records, files,
  users, pipelines or templates behave as "not found".
- **Record permissions** — the same visibility rules as the CRM and the REST API.
- **Strict input** — unknown arguments and unknown field names are rejected with the list of valid ones, values are
  validated against the field type (pick list, number, date, e-mail, rating, length, required).
- **Soft delete** — `delete_record` moves the record to the CRM trash.
- **No user editing** — there is no tool that changes users, roles or API keys.
- **Audit log** — every tool call is logged (tool name, argument *names*, affected record ids, result). Argument
  values are never logged.

### Prompt injection

Notes, e-mails and descriptions stored in the CRM are written by third parties. The server tells the model to
treat them as **data, never instructions**, and to ask the user before deleting or bulk-modifying records. Keep
human confirmation on for `delete_record` and the bulk tools in your client.

---

## ⏱️ Rate limits

| Limit | Default | Applies to |
|---|---|---|
| Requests | **300 per 5 minutes** per API key | every MCP HTTP request, including `initialize` and `tools/list` |
| Bulk writes | **10 calls per 5 minutes** (≤ 500 records) | `create_records` + `update_records` together |
| Document generation | **20 calls per 5 minutes** | `generate_document` |
| Failed authentication | 300 per 5 minutes per IP | `401` responses |

The request limit returns HTTP `429`; the per-tool limits return a tool error such as
`Too many bulk record writes, retry in 42 s.` A bulk call refused by the limit writes nothing.

### Other limits

| Item | Limit |
|---|---|
| Records per page | 50 (default 20) |
| Ids per `get_records_by_ids` | 50 |
| Items per bulk call | 50 |
| Files per write | 10, each ≤ 5 MB decoded |
| `download_file` / `generate_document` | ≤ 5 MB per file |
| Filter conditions | 20 per `advancedFilter` |
| Sort keys | 3 |
| Free-text `query` | 2–100 characters, scans at most 500 candidates |

---

## ❌ Errors

Tool failures are returned as a normal tool result with `isError: true` and a short, actionable message — it says
what was wrong and which tool to call next. Messages never contain stack traces. Typical prefixes:

| Prefix | Meaning |
|---|---|
| `VALIDATION_ERROR:` | Bad input (wrong type, invalid NIP checksum, `query` rules…) |
| `RESOURCE_NOT_FOUND:` | Record, file, user, pipeline or template not found (also for other companies' ids) |
| `FORBIDDEN:` / `UNAUTHORIZED:` | The integration user may not perform the operation |
| `INTERNAL_ERROR:` | Server-side failure — retry later |

Many validation messages have no prefix, e.g. `Unknown field(s) in \`data\`: foo. Valid fields: …`,
`Field 'status' (Status) must be one of: …`, `Missing required field(s): …`.

Bulk tools answer per item (`{ index, ok, id }` / `{ index, ok: false, error }`) and set `isError` only when every
item failed.

---

## 📚 See also

- [MCP Tools reference](tools.md)
- [Agent skill for InflowCRM](skill.md)
- [REST API — Modules & Fields](../rest-api/modules.md)
