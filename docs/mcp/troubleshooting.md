# MCP Troubleshooting

Find what you see, then follow the fix. Setup steps for each app are in [Connect your AI tool](clients.md).

**Jump to:** [Adding the connection](#adding-the-connection) · [Signing in](#signing-in) ·
[Connection errors](#connection-errors) · [Using the assistant](#using-the-assistant) · [Getting help](#getting-help)

---

## Adding the connection

### Gemini does not show "Custom apps"

Set Gemini to **English**: open [gemini.google.com/customize/connectors?hl=en](https://gemini.google.com/customize/connectors?hl=en).
Custom apps are visible only in English. This works in Poland too, including on the free plan. If the option is still
missing, check that you use a personal Google account with Keep Activity on.

### Claude does not let me add another connector

The Claude Free plan allows **one** custom connector. Remove the one you have, or use a paid plan. On Team and Enterprise
an owner adds the connector in **Organization settings → Connectors**, and you click **Connect** in
**Customize → Connectors**.

### ChatGPT or Perplexity does not offer a custom MCP server

The option depends on your plan and, in a company workspace, on what the administrator allows. In Perplexity
organizations an administrator must turn on **Allow members to add custom connectors**. In ChatGPT, workspace permissions or
Lockdown may block custom servers.

### My app only offers SSE

The server speaks Streamable HTTP only. If the app has a transport setting, choose **Streamable HTTP** or **HTTP**.
Apps that support only SSE cannot connect.

---

## Signing in

### "Nie znaleziono CRM o tym adresie albo MCP nie jest w nim włączone."

*(CRM not found at this address, or MCP is not enabled in it.)* Enter the address you use to log in to the CRM, e.g.
`firma.inflowstore.pl` (capital letters do not matter). If the address is right, [ask InflowCRM](../contact.md) to enable MCP for your company.

### "To konto nie może łączyć aplikacji AI z CRM. Skontaktuj się z administratorem firmy."

*(This account cannot connect AI apps to the CRM.)* An employee sees this when MCP for employees is not enabled for the
company yet, or when the employee has no role. [Ask InflowCRM](../contact.md) to enable MCP for employees; the
assistant then works within the permissions of the employee's own CRM role. Integration users connect with their API key.

### "Ten link wygasł. Wróć do aplikacji i połącz się jeszcze raz."

*(This link has expired.)* The sign-in link is valid for 10 minutes. Go back to your AI app and click **Connect** again.

### The consent screen warns about an unknown address

> *Ten adres nie należy do znanych aplikacji AI (claude.ai, chatgpt.com). Zezwól tylko wtedy, gdy to Ty łączysz tę aplikację.*

The address the sign-in returns to is not on our list of known apps. This is expected for Gemini, Perplexity and Grok.
Check that you started the connection yourself and that the address shown belongs to your AI app, then allow it. If you
did not start it, click deny.

### Claude Code shows "Needs authentication"

Run `/mcp`, select `inflowcrm` and choose **Authenticate** (or **Re-authenticate**). On a machine without a browser, use
`claude mcp login inflowcrm --no-browser`. If you configured an API key, a rejected key shows as `Failed to connect`
instead: check the key, then see [401](#_401-unauthorized).

---

## Connection errors

### 401 Unauthorized

The key or sign-in is missing, unknown or expired, the user is blocked or deleted, or the company account is inactive. Check
the key was copied whole and is sent as `Authorization: Bearer YOUR_API_KEY`. Signed-in apps: disconnect and connect again.

### 403 "MCP is not enabled for this company"

The key (or sign-in) is valid, but MCP is not enabled for your company. The answer carries
`error.code: "mcp_disabled"` and the message `MCP is not enabled for this company. Ask your CRM administrator or InflowCRM support to enable it.`
[Ask InflowCRM](../contact.md) to enable it. Earlier versions answered `404 Not Found` here.

### 404 Not Found

Rare and temporary: the server could not read your company's settings. Try again; if it persists,
[contact InflowCRM](../contact.md).

### 405 Method Not Allowed

Opening `https://mcp.inflowcrm.pl` in a browser, or any `GET`, gives this. It does not mean the server is down: MCP
accepts `POST` only, and your AI app handles that for you.

### 403 and "Forbidden origin"

The request came from a web page. Call MCP from a server-side client, not from browser code.

### 429 or "Too many …, retry in N s."

A rate limit (see [MCP Server](README.md)) was reached. Wait for the time given. A refused call writes nothing. The request
limit is shared by everything using the same API key, so give each integration its own integration user.

---

## Using the assistant

### Tools are missing

- Tools your plan, the enabled features or your account do not allow are not listed.
- After InflowCRM enables a feature, **reconnect** the app or start a new chat. Claude Code: `/mcp`, then reconnect.
- Check the connector is switched on for this chat, and that the tool is not switched off in your app's settings.
- When you signed in as an **employee**, the list follows your CRM role: a tool the role has no right for is not
  listed (for example no `delete_record` without the right to delete, no `create_records` without bulk import). Ask
  your company administrator to change the role — see [Tools depend on the role](tools.md#tools-depend-on-the-role).
- Notes are read with `list_record_notes` on the record they belong to; `search_records` does not take notes.

### The assistant says a field does not exist, or cannot see it

Only fields with an **Api field name** can be read, filtered and written. Set one in the field settings in the CRM, then
ask the assistant to read the fields again.

When you signed in as an **employee**, fields your CRM role cannot read — and fields marked hidden — are not shown to the
assistant at all, and it answers `Unknown field(s)` if it names one. Fields your role may only read are refused with
`PERMISSION_DENIED` when written, also when creating a record. Ask your company administrator for access.

### `PERMISSION_DENIED` or `FORBIDDEN`

The connected account may not do this. Ask your company administrator. There is no way around it from the assistant.

### The assistant keeps asking for confirmation, or a tool refuses without `confirm: true`

Sensitive changes need your agreement in the conversation, and your AI app may ask as well.
See [Confirmations](README.md#confirmations).

### A configuration change is refused with `VALIDATION_ERROR`

The message says what to fix, and nothing was changed. The usual ones:

- **Record numbering** — `update_numbering_settings` refuses a `format` without `{number}` exactly once or with an
  unknown token, a `resetPeriod` other than `none` whose `format` lacks the date tokens that tell the periods apart
  (numbers would repeat), and any change of `format`, `prefix`, `suffix`, `numberLength` or `resetPeriod` once the module
  has issued numbers — only `enabled` and `startingValue` can still be changed over MCP; the rest is done in the CRM
  application. `backfill_numbering` is refused while numbering of the module is off
  (`Numbering of '…' is switched off…`): switch it on first.
- **White-label** — `update_branding` answers `white-label is not active for this company; ask InflowCRM support` when
  the feature is off (`isActive: false` in `get_branding`). [Ask InflowCRM](../contact.md) to switch it on; the assistant
  cannot. Logo and favicon are never changed over MCP.
- **Company data** — a NIP with a wrong checksum, a knowledge base link that is not `https://`, a start page
  (`desktopRoute`) that is not in `allowedDesktopRoutes` of `get_company_settings`.
- **Module names** — a module that `list_modules` does not show as `renamable: true`, an empty or too long form
  (60 characters), or a module without default names (e.g. `contract`) given fewer than six forms the first time.
- **System fields** — `update_field` and `delete_field` refuse a built-in field of the CRM: it cannot be deleted, its
  type, Api field name or built-in status cannot be changed, and a required one cannot be hidden or made optional.
  Hide a non-required system field instead, or change it in the CRM.

Fix the value from the message; do not look for another way to get the same change in.

### A generated PDF does not reach me

Some AI apps (Claude, for example) cannot pass a large PDF from `generate_document` on to you. The answer then carries a
`hint`: ask the assistant to generate the document again with `saveToRecord: true` and open the file in the **Files**
tab of the record in InflowCRM. The PDF saved to the record needs the `files:write` scope, as always.

### A search misses a record, or times out

A search answer with `matchScope: "word-start"` checked only the beginnings of words, so it can list some records and
still miss one that has the text in the middle of a word. Ask the assistant to search again with
`matchScope: "anywhere"` before it decides a record does not exist or creates a duplicate. A search that times out did not
finish, and that does not mean the record is missing: do not create one on that basis. After `The text search took too long — narrow the query (more characters) and retry.` use a longer search text.
After `Search did not finish: the mid-word scan timed out. This does NOT mean no record exists. Do not retry right away; do not create a record on this result.` do not retry right away.
`Too many text searches, retry in N s.` means more than 30 slow searches in a minute for the company: wait for the time given.

### A count or report looks too low

An answer with `truncated: true` found 500 or more candidates, so its numbers are lower bounds: use a longer search text.
Lists come in pages of up to 50, so ask the assistant to count instead of reading pages.

### The assistant did something I did not ask for

Disconnect the app, then tell your company administrator. A record removed with `delete_record` is in the CRM trash.
Text inside CRM data should never steer the assistant, so [tell us](../contact.md) if it seems to have done.

---

## Getting help

[Contact InflowCRM](../contact.md) and include your CRM address, your AI app and plan, the time, and the exact message.
**Never send your API key**, and leave customer data out of screenshots.
