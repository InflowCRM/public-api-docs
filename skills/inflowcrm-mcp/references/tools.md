# MCP Tools Reference

All tools of the InflowCRM MCP server: up to **37**, plus **13 configuration tools** when they are enabled for your
company (up to 50). Tools your plan, the enabled features or your account do not allow are not listed at all —
e.g. pipeline tools appear only when your plan includes potentials or orders. See [MCP Server](README.md) for
connecting, authentication and limits.

**Conventions**

- `module` is a system module name returned by `list_modules` (e.g. `customer`, `potential`).
- Field keys are **Api field names** from `get_module_fields`. Fields without an Api field name are listed there
  without one and cannot be read, filtered or written.
- Ids are 24-character hex strings (`recordId`, `userId`, `fileId`, `templateId`, pipeline `process`); fields are
  identified by `fieldId` (e.g. `cf12`), list views by `listId`, roles by `roleId`.
- Unknown top-level arguments are rejected (input validation error). Every record in an answer has an `id`.
- Read-only tools are marked with the MCP `readOnlyHint`; the delete tools and `update_role` carry `destructiveHint`.
- Tools that need `confirm: true` refuse to run without it — set it only after the user agreed.
- A signed-in (OAuth) user works within their role; the API key acts as the integration user. A module the role cannot
  see is not in `list_modules`, its templates are not listed, and a tool naming it answers
  `PERMISSION_DENIED: Your CRM role does not allow this operation on module '<module>'. Retrying will not help: tell the user to ask their CRM administrator for access.`
  The same text answers a write the role does not allow. For a signed-in user `get_module_fields` and `get_template_tags`
  also leave out fields hidden by a field rule and, in the tags, relations and loops to modules the role cannot see. A
  role that may write a module but not read it gets
  `VALIDATION_ERROR: The data was refused. Your CRM role cannot read the fields of this module, so the details cannot be shown.`

---

## Discover

### `list_modules`
Lists the modules available to the caller (for a signed-in user: the ones their role can see) with the singular/plural names configured by the company. **Start here.**

- **Input:** none
- **Output:** `[{ moduleName, singular, plural, systemModuleName }]` — pass `systemModuleName` as `module`.

### `get_module_fields`
Fields of a module: `apiFieldName`, `label`, `type`, `required`, `pickListValues`, `relatedModule`, `relationField`,
`relationFieldName`, `length`, `decimal`, `defaultValue`, `description`. Call it before searching, filtering, creating or updating.

- **Input:** `{ module }`

### `get_sample_record`
One existing record of the module, to show the real shape of the data. `null` when the module is empty.

- **Input:** `{ module }`

---

## Read records

### `search_records`
Searches records with an optional free-text `query`, filters, sorting and paging.

| Argument | Type | Notes |
|---|---|---|
| `module` | string | required |
| `query` | string | 2–100 characters; case-insensitive fragment over text fields (name, e-mail, phone, NIP, record number…) |
| `filters` | object | simple equality filters `{ apiFieldName: value }`, max 20 keys |
| `advancedFilter` | object | `{ operator: "AND" \| "OR", conditions: [{ field, condition, value }] }`, 1–20 conditions |
| `sort` | object | `{ apiFieldName: 1 \| -1 }`, max 3 keys. Default: newest first |
| `page` | integer | 1-based, default 1 |
| `limit` | integer | 1–50, default 20 |
| `expandRelations` | boolean | relations as `{ id, label }` instead of bare ids |

**Output:** `{ records: [...], pagination: { page, limit, total, pages } }`, plus `truncated` when `query` is used.

### `find_record`
One record matching the filters, or `RESOURCE_NOT_FOUND`. Which record is returned is not defined when several
match — narrow the filters until the match is unique. Use it to check whether something exists before creating it.

- **Input:** `{ module, filters?, advancedFilter?, expandRelations? }`

### `count_records`
Number of records matching optional filters. Cheaper than `search_records` when only the number matters.

- **Input:** `{ module, filters?, advancedFilter? }` → `{ count }`

### `get_record`
- **Input:** `{ module, recordId, expandRelations? }`

### `get_records_by_ids`
Several records of one module (max 50 ids), paginated. Ids that do not exist are skipped.

- **Input:** `{ module, recordIds, sort?, page?, limit?, expandRelations? }` → same shape as `search_records`

---

## Filtering, search and sorting

**`filters`** — one value per field, matched for equality:

```json
{ "status": "Aktywny", "city": "Kraków" }
```

- A **date-only** value (`"2026-05-10"`) on a Date/Datetime field matches the whole day (server time zone); a value with a time matches exactly.
- A **DateRange** field takes `[from, to]` and matches records whose range **starts** between `from` and `to`:
  `{ "eventDate": ["2026-05-10", "2026-05-20"] }`. Leave `from` empty (`["", "2026-05-20"]`) for "starts up to";
  with only `from` it matches records starting on that day.
- `false` on a checkbox matches every record that is not `true` (including empty).
- Number/Rating fields accept numbers or numeric strings; `null` matches records with no value, `""` matches values
  stored as an empty string.
- To match one of several values use `advancedFilter` with `in`.

**`advancedFilter`** — conditions joined with `AND` (default) or `OR`:

```json
{
  "operator": "AND",
  "conditions": [
    { "field": "status", "condition": "in", "value": ["Nowy", "W trakcie"] },
    { "field": "createdTime", "condition": "greaterOrEqual", "value": "2026-01-01T00:00:00.000Z" },
    { "field": "email", "condition": "isNotEmpty" }
  ]
}
```

| Condition | Value |
|---|---|
| `equal`, `notEqual`, `greater`, `greaterOrEqual`, `lower`, `lowerOrEqual` | single value |
| `in`, `notIn` | array |
| `isEmpty`, `isNotEmpty` | none (`value: false` inverts) |
| `contains` | text, ≤ 200 characters; case-insensitive substring on text, e-mail, phone, NIP, link and pick list fields |

`advancedFilter` also accepts the system fields `id`, `createdTime`, `modifiedTime`, `createdBy`, `modifiedBy`,
`owner`, `recordNumber`. Unknown field names are rejected with the list of valid ones.

> Conditions on your own Date and Datetime fields take an ISO date or the field's date format; a value that is no date is
> `VALIDATION_ERROR: Wrong value type: …`. `equal` compares the exact moment: a date-only value (`2026-05-10`) matches
> what was saved as that date, not a Datetime later that day. For a whole day use `filters`, or `greaterOrEqual` with
> `lower`.

**Rules**

- `filters` and `advancedFilter` cannot be combined.
- `query` cannot be combined with `filters`; it can be combined with an `advancedFilter` using `AND`.
- `query` scans at most 500 matching candidates (it may also match text fields that have no Api field name).
  `truncated: true` means the cap was hit and totals may be
  capped — type a longer fragment; extra filter conditions do not shrink the scan.
- **Sort** keys: Api field names or `createdTime`, `modifiedTime`, `recordNumber`, `owner`, `createdBy`,
  `modifiedBy`. Default `{ "createdTime": -1 }`.

---

## Write records

### `create_record`
Creates a record and returns it with its new `id`. Required fields must be set.

- **Input:** `{ module, data, files? }`

### `update_record`
Updates only the fields listed in `data`; other fields stay unchanged. Files sent as `standaloneAttachments` are
appended to the record's files; a file sent for a File field replaces that field's value.

- **Input:** `{ module, recordId, data, files? }` — at least one field or one file.

### `create_records` / `update_records` (bulk)
1–50 records of one module per call; each item is `{ data }` (create) or `{ recordId, data }` (update). Items are
processed independently.

- **Output:** `{ results: [{ index, ok: true, id } | { index, ok: false, error }], succeeded, failed }`
- `isError` is set only when every item failed.
- No file uploads in bulk calls. Shared limit: 10 bulk calls per 5 minutes.

### `delete_record`
Moves a record to the CRM trash. → `{ id, deleted: true }`

- **Input:** `{ module, recordId }`

### `data` — field values

- Keys are Api field names; values are validated against the field type:
  - pick list / radio / multi pick list values must be one of `pickListValues` (unless the field is configured to
    accept new items),
  - numbers, dates (the field's date format or ISO `2026-05-10`), checkboxes (`true`/`false`),
  - e-mail format, NIP (10 digits with a correct checksum, or a VAT id with a country prefix), rating between 0 and the
    field's maximum (5 unless configured), text length (`length`).
- Unknown keys are rejected with the list of valid fields. Read-only system fields copied from a fetched record
  (`id`, `createdTime`, `modifiedTime`, `createdBy`, `modifiedBy`, `recordNumber`) are ignored, so a record can be read,
  changed and sent back — unless it was read with `expandRelations` (send relations back as ids) or holds old values
  that no longer pass validation.
- **Relations** take a record id (multi-relations an array of ids); `null` or `""` clears the relation. Ids of
  records that do not exist in your company are rejected. For a signed-in user a new id must also be a record their role
  can read; both cases answer `VALIDATION_ERROR: Field '<field>' references a record that does not exist`. A relation
  already stored on the record does not block changing other fields.
- **Owner:** a new record belongs to the user behind the connection (the integration user for an API key). To assign someone else, send a user id from `list_users`
  in the field whose `relationField` is `owner` (see `get_module_fields` for its Api field name).
- **Pipelines** (`potential`, `order`): set the pipeline id (from `get_pipelines`) and the **stage name** together.
  The stage is matched by name (`"1"` means a stage called "1", not the first stage). On create, or when moving a
  record to another pipeline, a missing stage means the first stage; an update that keeps the pipeline keeps its stage. Setting the pipeline field to `null` detaches the record and removes its card from the
  board (when the field is not required). Check `get_module_fields` for the Api field names of the pipeline and
  stage fields in your company.

### `files` — attachments in `create_record` / `update_record`

```json
{
  "files": [
    { "key": "standaloneAttachments", "name": "offer.pdf", "mimeType": "application/pdf", "contentBase64": "JVBERi0x..." }
  ]
}
```

- `key`: Api field name of a File field, or `standaloneAttachments` to attach the file to the record itself.
- Up to 10 files per call, each ≤ 5 MB decoded; standard or URL-safe base64. The whole request is limited to
  7 MB, so one call carries about 5 MB of files in total — send larger sets in separate `update_record` calls.
- Same extension / MIME / content checks as the REST API; mismatched or blocked types are rejected.

---

## Pipelines

### `get_pipelines`
Pipelines (sales processes) of `potential` or `order` with their stage names.

- **Input:** `{ module }` → `[{ id, systemLabel, stagesNames: [...] }]`

### `get_pipeline_summary`
Record count per stage and the sum of the value per currency — a funnel report without reading records.

- **Input:** `{ module, process? }` (`process` = pipeline id; default: all pipelines of the module)
- **Output:** `{ module, pipelines: [{ id, name, count, totals: [{ currency, value }], stages: [{ name, count, totals }] }] }`
- `totals` stay empty when the module has no value/currency fields; records with a value but no currency are summed
  under `currency: null`.

---

## Files

### `list_record_files`
- **Input:** `{ module, recordId }` → `[{ id, fileName, fileSize, mimeType, fileExtension, createdTime }]`

### `download_file`
Downloads a file (≤ 5 MB) attached to a record you can see. Larger files: open them in InflowCRM.

- **Input:** `{ fileId }`
- **Output:** text item `{ id, fileName, mimeType, fileSize }` + an `image` item (images) or an embedded `resource`
  with a base64 `blob` (other files).

---

## Users

### `list_users`
Active users of the company: `id`, `systemLabel` (name), `email` and other fields with Api field names. Use the
ids to set `owner` or user relations. No passwords or API keys are ever returned. A signed-in employee sees the
users and fields the CRM's user list shows them.

### `get_user`
- **Input:** `{ userId }`

---

## Documents

### `list_document_templates`
PDF templates the company can generate. To read or change a template's content use the
[document template tools](#document-templates).

- **Input:** `{ module? }` → `[{ id, name, module }]`

### `generate_document`
Renders a template for one record into a PDF (≤ 5 MB) and returns it. Limit: 20 calls per 5 minutes.

- **Input:** `{ templateId, recordId, saveToRecord? }` — the record must belong to the template's module.
  `saveToRecord: true` also stores the PDF as a file of the record.
- **Output:** text item `{ fileName, fileSize, fileId? }` + an embedded PDF `resource` (base64).
- A signed-in user needs read access to every related record the template prints. Otherwise:
  `PERMISSION_DENIED: This template prints a related record your CRM role cannot read, so the document cannot be generated for this record. Tell the user to ask their CRM administrator for access or to use another template.`
  A template of a module their role cannot see answers `RESOURCE_NOT_FOUND: Template not found…`.

---

## Templates

SMS, e-mail and PDF document templates of the company. These tools **prepare templates — they never send messages**;
sending stays in the CRM (manually or by a template's automatic sending, which these tools cannot switch on or off).

**Common rules**

- `module`: the module the template belongs to (from `list_modules`). It is set on create and cannot be changed.
- `name`: 1–200 characters, `description`: up to 2,000 characters — both plain text (no HTML tags).
- `public`: visible to everyone in the company (default `true` on create). A non-public template of another user
  is "not found" for a user whose role does not let them see it.
- `templateId`: from the matching list tool. Ids of other companies' or deleted templates →
  `RESOURCE_NOT_FOUND: Template <id> not found. Call list_…_templates for valid ids.`
- A user whose role may not create, modify or delete templates gets
  `PERMISSION_DENIED: This API key's role is not allowed to … templates.`
- Creates, updates and deletes of all template kinds share one limit: **60 calls per 5 minutes**
  (`Too many template writes, retry in N s.`).
- Deleting a template moves it to the CRM trash and always needs `confirm: true`.
- `autoSending` (read only) shows `{ enabled, autoSendingEnabled }`; when both are `true` the CRM sends the template
  by itself, so changing its content needs `confirm: true` (see below).

### Tags

Templates insert record data with **tags** in single braces: `{CUSTOMER_NAME}`, related records as
`{RELATION.FIELD}`, built-ins such as `{DATE j-m-Y}`; e-mail and document templates can also loop over related records
with `{{#each …}}…{{/each}}`.

- A create or update with an unknown tag is refused:
  `Unknown tag(s) in <template|subject|header|footer>: {X}. Valid tags: … Call get_template_tags for the full list.`
  Tags already present in a stored template stay allowed on update.
- `{{…}}` expressions must not contain HTML (`<` or `>`).

### `get_template_tags`
Tags a template of a module may contain. Call it before writing a template.

- **Input:** `{ module, kind }` — `kind`: `sms` | `email` | `document`
- **Output:** `{ module, kind, fields: [{ tag, name, block, type }], relations: [{ index, name, relatedModule, hint? }], builtIns: [...], loops?: [{ tag, relatedModule }] }`
  (`loops` for e-mail and document templates only)

### SMS templates

#### `list_sms_templates`
- **Input:** `{ module? }` → `[{ id, name, module, description, public, autoSending }]`, sorted by name (no text — use `get_sms_template`).

#### `get_sms_template`
- **Input:** `{ templateId }` → `{ id, name, module, description, template, public, autoSending, senderName, createdTime, modifiedTime }`
  (`senderName` read only).

#### `create_sms_template`
- **Input:** `{ name, module, template, description?, public? }` — `template`: the SMS text, 1–1,000 characters.
- Created with automatic sending **off**. HTML in the text is removed when saved; the answer shows the stored text.
- **Output:** the template as in `get_sms_template`.

#### `update_sms_template`
- **Input:** `{ templateId, name?, description?, template?, public?, confirm? }` — at least one change
  (`Nothing to update: pass at least one of name, description, template, public.`).
- **`confirm: true`** is needed when the template is sent automatically and `template` changes:
  `This template is sent automatically (autoSending.enabled). Show the new text to the user and call again with confirm: true.`
  Name, description or visibility changes never need it.

#### `delete_sms_template`
- **Input:** `{ templateId, confirm: true }` → `{ id, deleted: true }`

### E-mail templates

#### `list_email_templates`
- **Input:** `{ module? }` → same shape as `list_sms_templates`.

#### `get_email_template`
- **Input:** `{ templateId }` → `{ id, name, module, description, subject, template, public, autoSending, senderName, attachedDocuments, attachedFiles, createdTime, modifiedTime }`
- `template` is the HTML body; `attachedDocuments` / `attachedFiles` are counts. Sender, attachments and automatic
  sending are managed in the CRM, not here.

#### `create_email_template`
- **Input:** `{ name, module, subject, template, description?, public? }` — `subject`: 1–500 characters, plain text;
  `template`: HTML body, ≤ 5 MB.
- Created with automatic sending off and no attachments.
- **The HTML is sanitized when saved:** scripts, event handlers, iframes and other unsafe markup are removed, ordinary
  formatting (paragraphs, lists, tables, links…) is kept. The answer contains the stored subject and body.

#### `update_email_template`
- **Input:** `{ templateId, name?, description?, subject?, template?, public?, confirm? }` — at least one change.
- **`confirm: true`** is needed when the template is sent automatically and `subject` or `template` changes (same
  message as for SMS).

#### `delete_email_template`
- **Input:** `{ templateId, confirm: true }` → `{ id, deleted: true }`

### Document templates

PDF templates rendered by `generate_document`. Find them with `list_document_templates` (see *Documents*).

#### `get_document_template`
- **Input:** `{ templateId }` → `{ id, name, module, description, template, header, footer, settings, public, createdTime, modifiedTime }`

#### `create_document_template`
- **Input:** `{ name, module, template, header?, footer?, settings?, description?, public? }`
  - `template`, `header`, `footer`: HTML, ≤ 5 MB each; sanitized when saved like e-mail HTML. `{PAGENO}` and
    `{nbpg}` print the page number and page count.
  - `settings`: `{ format?, orientation?, marginTop?, marginBottom?, marginLeft?, marginRight?, marginHeader?, marginFooter? }`
    — `format`: `A3` | `A4` | `A5` | `A6`; `orientation`: `P` (portrait) | `L` (landscape); margins in mm, 0–200.
    Defaults: A4, portrait, margins 20, header/footer margins 10.
- **Output:** the template as in `get_document_template`.

#### `update_document_template`
- **Input:** `{ templateId, name?, description?, template?, header?, footer?, settings?, public?, confirm? }` — at least
  one change. `""` removes the header or footer; settings you do not send keep their values.
- **`confirm: true`** is needed when `template`, `header` or `footer` changes and the document is attached to an
  automatically sent e-mail template:
  `This document is attached to automatically sent e-mail template(s): <names>. Show the change to the user and call again with confirm: true.`

#### `delete_document_template`
- **Input:** `{ templateId, confirm: true }` → `{ id, deleted: true }`

---

## Lookup

### `lookup_company_gus`
Official name and address of a Polish company from the GUS (REGON) registry.

- **Input:** at least one of `nip` (10 digits, checksum verified), `regon` (9–14 digits), `krs` (10 digits).
- Typical use: fill in a new customer from a NIP before `create_record`.

---

## Configuration tools

Fields, saved list views and roles — **only when InflowCRM has enabled the configuration tools for your company**, and
only for a company administrator (signed in) or an API key; an employee never gets them. Every change applies to
**all users of the company** at once.

**Common rules**

- Creates, updates and deletes of fields, list views and roles share one limit: **30 calls per 5 minutes**
  (`Too many configuration changes, retry in N s.`).
- `delete_field`, `delete_list_view`, `update_role` and `delete_role` require `confirm: true` — send it only after the
  user agreed. For several changes, ask once for the whole batch. Without it:
  `VALIDATION_ERROR: This change needs confirm: true. Ask the user to approve it, then call again with confirm: true.`
  The refused call is recorded in the audit log and does not count against the limit.
- Ids of other companies or deleted items → `RESOURCE_NOT_FOUND: … does not exist.` with the tool to call next.
- Deletes are soft: the item disappears from the CRM, stored record values are kept.

### Fields and pick lists

#### `get_field_layout`
The module's fields as the CRM settings see them, block by block — including hidden and system fields and what may
be changed.

- **Input:** `{ module }`
- **Output:** `{ module, blocks: [{ name, hidden, fields: [...] }] }`; each field: `fieldId`, `apiFieldName`, `label`,
  `type`, `required`, `hidden`, `system`, `locks: { delete, hide, required, pickListValues }`, `description`,
  `pickListValues`, `pickListColors`, `allowNewItems`, `defaultValue`, `length`, `decimals`, `maxRating`,
  `relatedModule`, `showInList`, `showInCreateForm`.

#### `create_field`
Adds a custom field. Its type and Api field name cannot be changed later.

| Argument | Type | Notes |
|---|---|---|
| `module` | string | required |
| `label` | string | required, 1–100 characters, unique in the module |
| `type` | string | required: `String`, `Number`, `Date`, `Datetime`, `PickList`, `MultiPicklist`, `RadioButton`, `CheckBox`, `Email`, `Phone`, `Link`, `Nip`, `Rating` |
| `block` | string | block name; default: the first visible block |
| `apiFieldName` | string | letters and digits, starts with a letter, ≤ 64; generated when omitted |
| `description` | string | ≤ 500 |
| `required` | boolean | default `false` |
| `pickListValues` | string[] | required for `PickList`, `MultiPicklist`, `RadioButton` (1–200 values, unique) |
| `pickListColors` | object | `{ value: "#RRGGBB" }` |
| `allowNewItems` | boolean | pick lists only |
| `defaultValue` | string / number / boolean / string[] | must fit the type and the pick list values |
| `length` | integer | `String` only, 1–10,000 |
| `decimals` | integer | `Number` only, 0–10 |
| `maxRating` | integer | `Rating` only, 1–10 (default 5) |
| `showInList`, `showInCreateForm` | boolean | default `false` |

- **Output:** `{ module, field }`
- Typical errors: `CONFLICT: A field labelled '…' already exists in this module…`, `CONFLICT: apiFieldName '…' is
  already used…`, `VALIDATION_ERROR: Block '…' does not exist in this module. Available blocks: …`

#### `update_field`
Changes one field. Send only what changes.

- **Input:** `{ module, fieldId, label?, description?, required?, hidden?, pickListValues?, addPickListValues?, removePickListValues?, pickListColors?, allowNewItems?, defaultValue?, length?, decimals?, maxRating?, showInList?, showInCreateForm? }`
- **Pick lists:** either `pickListValues` (the full new list) **or** `addPickListValues` / `removePickListValues`, not
  both. A pick list keeps at least one value. Colors are merged; `""` clears a color.
- **Removing a value never changes records:** the answer lists `removedPickListValues: [{ value, recordsStillUsing }]`
  and a `note` — update those records if the old value should disappear.
- `hidden: true` also clears `required`. `defaultValue: null` clears the default.
- Refused: hiding or making optional a required system field; changing pick lists that drive pipelines, stages or
  statuses (change those in the CRM); a label another field already has.
- **Output:** `{ module, field, removedPickListValues?, note? }`

#### `delete_field`
Deletes one or more custom fields of one module — **all or nothing**: one unknown or system field refuses the whole
call. The fields disappear from forms and lists, records keep their values, and the Api field name stays reserved.

- **Input:** `{ module, fieldIds, confirm: true }` — `fieldIds`: 1–25 `fieldId`s from `get_field_layout`.
- **Output:** `{ deleted: true, fields: [{ fieldId, apiFieldName, recordsWithValue, listViewsUsingField }] }`
- System fields cannot be deleted (`… is a system field and cannot be deleted; hide it with update_field (hidden: true) instead.`);
  required system fields cannot be hidden either (`… is a required system field of <module> and cannot be removed at all.`).
- Delete several fields of a module in **one** call after one confirmation — not one call per field.

### Saved list views

#### `get_list_views`
All saved list views of the company, whoever may see them.

- **Input:** `{ module? }`
- **Output:** up to 200 views `[{ listId, name, module, columns, conditions: { and: [...], or: [...] }, sorting, visibility: { public, roleIds, userIds, squadIds }, visibleAsButton }]`

#### `create_list_view`

| Argument | Type | Notes |
|---|---|---|
| `module` | string | required |
| `name` | string | required, 1–100, unique in the module |
| `columns` | string[] | required, 1–50 Api field names, or `recordNumber`, `createdBy`, `modifiedBy`, `createdTime`, `modifiedTime` |
| `conditions` | object | `{ and?: [...], or?: [...] }`, each ≤ 30 `{ field, operator, value? }` |
| `sorting` | object | `{ field, direction: "asc" \| "desc" }` |
| `public` | boolean | default `false` |
| `roleIds`, `userIds`, `squadIds` | string[] | who sees the view (≤ 100 each) |
| `visibleAsButton` | boolean | default `false` |

- **Audience:** a view must be `public: true` or name at least one role, user or squad.
- **Operators by field type:**

| Field type | Operators |
|---|---|
| Text, e-mail, phone, NIP, link | `equal`, `notEqual`, `contain`, `notContain`, `startsWith`, `emptyString`, `notEmptyString` |
| Number, rating | `equalNumber`, `greater`, `greaterOrEqual`, `less`, `lessOrEqual`, `emptyNumber`, `notEmptyNumber` |
| Pick list, radio | `equal`, `notEqual`, `emptyString`, `notEmptyString` |
| Multi pick list | `includes`, `notIncludes`, `emptyString`, `notEmptyString` |
| Date, datetime | `equalDate`, `greaterDate`, `greaterOrEqualDate`, `lessDate`, `lessOrEqualDate`, `emptyDate`, `notEmptyDate`, `today`, `yesterday`, `tomorrow`, `thisWeek`, `lastWeek`, `nextWeek`, `thisMonth`, `lastMonth`, `nextMonth`, `thisYear`, `lastYear`, `nextYear`, `previous7days`, `next7days`, `nextDays`, `prevDays`, `futureDate`, `pastDate`, `year` |
| Date range | `emptyDate`, `notEmptyDate`, `dateContains`, `dateStartsAt`, `dateEndAt` |
| Checkbox | `isTrue`, `isFalse` |
| File | `emptyString`, `notEmptyString` |
| Relation | `equal`, `notEqual` (value: a record id) |
| Multi relation | `includes`, `notIncludes` |

- Operators like `emptyString`, `today` or `isTrue` take no `value`; the others take **one** value (no arrays). Dates:
  ISO date `2026-11-02` or a timestamp with time zone.
- **Output:** the created view. Name taken → `CONFLICT: A list view named '…' already exists in module '…'…`

#### `update_list_view`
- **Input:** `{ listId, name?, columns?, conditions?, sorting?, public?, roleIds?, userIds?, squadIds?, visibleAsButton? }`
- A property you send **replaces** the stored one entirely — read the view first and send the full new value.
  `conditions: {}` clears the filter, `sorting: null` removes the sort. The module cannot change.
- After a field used by the view is deleted, `get_list_views` shows it as `{ field: null, fieldId }`. Sent back, such a
  column or sort is dropped. A condition on it is refused with `VALIDATION_ERROR`: leave it out once the user agreed,
  because the view then shows more records.

#### `delete_list_view`
- **Input:** `{ listId, confirm: true }` → `{ deleted: true, listId, name, module }`

### Roles

#### `list_roles`
- **Input:** none → `[{ roleId, name, userCount }]`

#### `get_role`
- **Input:** `{ roleId }` → `{ roleId, name, userCount, modules: [{ module, see, create, modify, delete, … }], customPermissions }`
- Module rights: `see`, `share`, `create`, `modify`, `delete`, `seeOthers`, `modifyOthers`, `deleteOthers`,
  `seeSquad`, `modifySquad`, `modifyShared`, `deleteSquad`, `deleteShared`, `seeRole`, `modifyRole`, `deleteRole`,
  `massEdit`, `massDelete`, `massExport`, `massImport`. Modules marked `derivedFrom` are computed from
  `customPermissions` and are changed there.

#### `create_role`
- **Input:** `{ name, copyFromRoleId?, modules?, customPermissions? }` — `name` unique; without `copyFromRoleId` the
  role starts from the CRM defaults, with it from a copy of that role (including its field rules).
- `modules`: `[{ module, <right>: true | false, … }]`, each module once; `customPermissions`: e.g.
  `canSendEmail`, `canSendSms`, `canSeeEmail` (`none` | `see` | `seeSquad` | `seeOthers`), `canEditEmailTemplate`
  (`none` | `modify` | `modifyShared` | `modifySquad` | `modifyOthers`)…
- Rights to manage users can be granted only in the CRM; over MCP the `user` module accepts only `see`, `seeOthers`,
  `seeSquad`, `seeRole` (its other rights may only be revoked). Rights to manage roles, fields, field rules, company
  settings and squads are also granted only in the CRM. Sending any right of a `derivedFrom` module is refused — set
  the matching `customPermissions` key instead.
- **Output:** the new role (`userCount: 0`).

#### `update_role`
Changes the name, module rights (only the rights you send) or custom permissions (only the keys you send) of a role.
Every user holding the role is affected — the answer's `userCount` says how many.

- **Input:** `{ roleId, confirm: true, name?, modules?, customPermissions? }`
- **Output:** the role after the change.

#### `delete_role`
- **Input:** `{ roleId, confirm: true }` → `{ deleted: true, roleId, name, listViewsReferencingRole }`
- Refused while users have the role:
  `CONFLICT: 2 users still have this role; move them to another role in the CRM user settings first, then delete the role.`
  MCP does not edit users — move them in the CRM's user settings.
