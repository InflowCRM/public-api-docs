# MCP Tools Reference

Full reference of the InflowCRM MCP tools. Setup, authentication and rate limits: https://inflowcrm.github.io/public-api-docs/#/docs/mcp/README

**Conventions**

- `module` is a system module name returned by `list_modules` (e.g. `customer`, `potential`).
- Field keys are **Api field names** from `get_module_fields`. Fields without an Api field name are listed there
  without one and cannot be read, filtered or written.
- Ids are 24-character hex strings (`recordId`, `userId`, `fileId`, `templateId`, pipeline `process`).
- Unknown top-level arguments are rejected (input validation error). Every record in an answer has an `id`.
- Read-only tools are marked with the MCP `readOnlyHint`; `delete_record` carries `destructiveHint`.

---

## Discover

### `list_modules`
Lists the modules available to the API key with the singular/plural names configured by the company. **Start here.**

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

> Range conditions (`greater`, `lower`, …) on dates are supported for `createdTime` and `modifiedTime`. For your own
> Date fields use `filters` (a whole day) or a DateRange field.

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
  - e-mail format, rating between 0 and the field's maximum (5 unless configured), text length (`length`).
- Unknown keys are rejected with the list of valid fields. Read-only system fields copied from a fetched record
  (`id`, `createdTime`, `modifiedTime`, `createdBy`, `modifiedBy`, `recordNumber`) are ignored, so a record can be read,
  changed and sent back — unless it was read with `expandRelations` (send relations back as ids) or holds old values
  that no longer pass validation.
- **Relations** take a record id (multi-relations an array of ids); `null` or `""` clears the relation. Ids of
  records that do not exist in your company are rejected.
- **Owner:** a new record belongs to the integration user. To assign someone else, send a user id from `list_users`
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
ids to set `owner` or user relations. No passwords or API keys are ever returned.

### `get_user`
- **Input:** `{ userId }`

---

## Documents

### `list_document_templates`
PDF templates the company can generate.

- **Input:** `{ module? }` → `[{ id, name, module }]`

### `generate_document`
Renders a template for one record into a PDF (≤ 5 MB) and returns it. Limit: 20 calls per 5 minutes.

- **Input:** `{ templateId, recordId, saveToRecord? }` — the record must belong to the template's module.
  `saveToRecord: true` also stores the PDF as a file of the record.
- **Output:** text item `{ fileName, fileSize, fileId? }` + an embedded PDF `resource` (base64).

---

## Lookup

### `lookup_company_gus`
Official name and address of a Polish company from the GUS (REGON) registry.

- **Input:** at least one of `nip` (10 digits, checksum verified), `regon` (9–14 digits), `krs` (10 digits).
- Typical use: fill in a new customer from a NIP before `create_record`.
