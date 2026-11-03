# Agent Skill for InflowCRM

The MCP server gives an assistant the **tools**. The InflowCRM **agent skill** gives it the **know-how**: which
tool to call first, how CRM fields and pipelines work, how to filter without trial and error, and when to stop and
ask before changing data. With the skill the assistant needs fewer calls and makes fewer mistakes.

Source: [`skills/inflowcrm-mcp`](https://github.com/InflowCRM/public-api-docs/tree/main/skills/inflowcrm-mcp)
— `SKILL.md` plus a full tool reference in `references/tools.md`.

> The skill does not connect anything by itself. Connect the [MCP server](README.md) first — or paste the
> [universal setup prompt](clients.md#universal-setup-prompt) into your agent to do both in one go.

---

## What the skill teaches

- **Discovery first** — `list_modules` → `get_module_fields`, and use only Api field names as keys.
- **Choosing the right read** — free-text `query` for names/e-mails/NIP, `filters` / `advancedFilter` for exact
  criteria, `count_records` and `get_pipeline_summary` instead of paging through records.
- **Data conventions** — date and DateRange filters, pick list values, relations by id, `expandRelations`.
- **Pipelines** — pipeline id and stage *name* travel together; stages are matched by name, not position.
- **Safe writes** — duplicate check before create, GUS lookup for new Polish companies, explicit user confirmation
  before deletes and bulk changes, CRM text is never treated as instructions.
- **Templates** — read first, tags only from `get_template_tags`, drafts shown to the user, extra care for
  templates the CRM sends automatically; the tools prepare templates and never send messages.
- **CRM configuration** (when enabled) — fields and pick lists, list views and roles: read before change, show
  before/after and how many users a role change affects, one confirmation for a whole batch (one `delete_field`
  call for several fields), `confirm: true` only after the user agreed, users with a role are moved in the CRM.
- **Recovering from errors** — what each validation message means and how to fix the call.

---

## Install

### Claude Code — plugin (recommended)

```text
/plugin marketplace add InflowCRM/public-api-docs
/plugin install inflowcrm@inflowcrm
```

Or from a terminal:

```bash
claude plugin marketplace add InflowCRM/public-api-docs
claude plugin install inflowcrm@inflowcrm
```

Update later with `/plugin marketplace update inflowcrm`.

### Claude Code — copy the folder

```bash
git clone --depth 1 https://github.com/InflowCRM/public-api-docs.git /tmp/inflowcrm-docs
mkdir -p ~/.claude/skills
cp -r /tmp/inflowcrm-docs/skills/inflowcrm-mcp ~/.claude/skills/
```

Use `.claude/skills/` inside a project instead of `~/.claude/skills/` to share it with your team via git.

### Copy and paste

Prefer not to install anything? Copy the skill below (the copy button is in the top-right corner of the block):

- **Claude Code:** save it as `~/.claude/skills/inflowcrm-mcp/SKILL.md` (or `.claude/skills/inflowcrm-mcp/SKILL.md` in a project).
- **Any other assistant:** paste it into the project instructions or system prompt.

The tool-by-tool details live in [`references/tools.md`](https://github.com/InflowCRM/public-api-docs/blob/main/skills/inflowcrm-mcp/references/tools.md)
— the skill works without it, the [MCP Tools](tools.md) page has the same content.

[SKILL.md](../../skills/inflowcrm-mcp/SKILL.md ':include :type=code markdown')

### Other agents

The skill is plain Markdown in the Agent Skills format (`SKILL.md` with `name` and
`description` front matter). Agents that support skills can load the folder as is; for others, add the content of
`SKILL.md` to the system prompt or project instructions.

---

## Try it

With the MCP server connected and the skill installed, ask for example:

- *"Show me the 10 newest customers from Kraków with their owners."*
- *"How many potentials are in each stage of our sales pipeline, and what is their total value?"*
- *"Add a customer from NIP 1234563218 and create a follow-up task for me next Monday."*
- *"Move the potential 'Szkolenie BHP — ACME' to stage 'Oferta wysłana'."*
- *"Generate the offer PDF for order 1042 and attach it to the order."*
- *"Draft an e-mail template for customers with a reminder about the training date."*
- *"Add a pick list field 'Źródło leada' to customers with the values Polecenie, Strona www, Targi."* (configuration
  tools)
