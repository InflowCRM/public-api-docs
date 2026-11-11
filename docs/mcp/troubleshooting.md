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

The sign-in pages can appear in Polish: the page asking for your CRM address is always in Polish, and the consent screen
follows your CRM's language. Each message below is given in English, followed by the Polish text you may see.

### "CRM not found at this address, or MCP is not enabled for it."

On screen: *Nie znaleziono CRM o tym adresie albo MCP nie jest w nim włączone.*

Enter the address you use to log in to the CRM, e.g.
`firma.inflowstore.pl`. If the address is right, [ask InflowCRM](../contact.md) to enable MCP for your company.

### "This account cannot connect AI apps to the CRM. Contact your company administrator."

On screen: *To konto nie może łączyć aplikacji AI z CRM. Skontaktuj się z administratorem firmy.*

For now MCP is available to company administrators only.
We will soon add an option that lets employees use MCP within the permissions of their own CRM role.

### "This link has expired. Go back to the app and connect again."

On screen: *Ten link wygasł. Wróć do aplikacji i połącz się jeszcze raz.*

The sign-in link is valid for 10 minutes. Go back to your AI app and click **Connect** again.

### The consent screen warns about an unknown address

> *This address does not belong to a known AI app (claude.ai, chatgpt.com). Allow only if you are the one connecting this app.*

On screen: *Ten adres nie należy do znanych aplikacji AI (claude.ai, chatgpt.com). Zezwól tylko wtedy, gdy to Ty łączysz tę aplikację.*

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

### 404 Not Found

The key is valid, but MCP is not enabled for your company. [Ask InflowCRM](../contact.md) to enable it.

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

### The assistant says a field does not exist, or cannot see it

Only fields with an **Api field name** can be read, filtered and written. Set one in the field settings in the CRM, then
ask the assistant to read the fields again.

### `PERMISSION_DENIED` or `FORBIDDEN`

The connected account may not do this. Ask your company administrator. There is no way around it from the assistant.

### The assistant keeps asking for confirmation, or a tool refuses without `confirm: true`

Sensitive changes need your agreement in the conversation, and your AI app may ask as well.
See [Confirmations](README.md#confirmations).

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
