# InflowCRM Public API – File Upload Guide

This guide explains how to upload files using the Public API, including multipart form-data usage, code examples, validation rules, and troubleshooting tips.

---

## 📦 Supported Endpoints

- `POST /api/createRecord`
- `PATCH /api/updateRecord/{id}`

File uploads are only supported for modules with file-type fields.
Maximum: **10 files per request**, **15MB per file**.

---

## 📝 Multipart Form-Data Example

> **Important:** All fields must be sent as top-level form fields. **Do not nest fields under `data`.**
> **Only fields with an 'Api field name' set in the InflowCRM UI can be used in the API.**

**Request (curl):**
```bash
curl -X POST "https://srv.inflowcrm.pl/api/createRecord" \
  -H "x-api-key: YOUR_API_KEY" \
  -F "module=customer" \
  -F "name=John Doe" \
  -F "email=john@example.com" \
  -F "profilePicture=@/path/to/photo.jpg"
```

**Request (JavaScript, fetch):**
```javascript
const form = new FormData();
form.append('module', 'customer');
form.append('name', 'John Doe');
form.append('email', 'john@example.com');
form.append('profilePicture', fileInput.files[0]);

fetch('https://srv.inflowcrm.pl/api/createRecord', {
  method: 'POST',
  headers: { 'x-api-key': 'YOUR_API_KEY' },
  body: form
});
```


---

## 📎 Standalone Attachments

If you need to attach files to a record **without** binding them to a specific custom field, use the reserved multipart key `standaloneAttachments`. Files uploaded under this key are persisted directly into the record's `files[]` array and are flagged as standalone (`isStandaloneFile: true`).

- The key `standaloneAttachments` is **reserved** and is treated as standalone regardless of any module schema. The verbose name is intentional — it avoids collisions with common user-defined `apiFieldName` values like `attachments` or `files`.
- You may repeat the `standaloneAttachments` key in a single request to upload **multiple files at once**.
- Standalone uploads can be **mixed** with named file-field uploads in the same request.
- On `PATCH /api/updateRecord/{id}`, newly uploaded files are **appended** to the record's existing `files[]` — they do not replace prior attachments.

**Standalone-only example (curl):**
```bash
curl -X POST "https://srv.inflowcrm.pl/api/createRecord" \
  -H "x-api-key: YOUR_API_KEY" \
  -F "module=customer" \
  -F "name=Acme Corp" \
  -F "standaloneAttachments=@/path/to/contract.pdf" \
  -F "standaloneAttachments=@/path/to/invoice.pdf" \
  -F "standaloneAttachments=@/path/to/notes.docx"
```

**Mixed example (named field + standalone, curl):**
```bash
curl -X POST "https://srv.inflowcrm.pl/api/createRecord" \
  -H "x-api-key: YOUR_API_KEY" \
  -F "module=customer" \
  -F "name=Acme Corp" \
  -F "profilePicture=@/path/to/avatar.jpg" \
  -F "standaloneAttachments=@/path/to/contract.pdf" \
  -F "standaloneAttachments=@/path/to/invoice.pdf"
```

**Update example (curl, appends to existing files):**
```bash
curl -X PATCH "https://srv.inflowcrm.pl/api/updateRecord/$RECORD_ID" \
  -H "x-api-key: YOUR_API_KEY" \
  -F "module=customer" \
  -F "standaloneAttachments=@/path/to/extra.pdf"
```

> **Note:** the reserved key wins over any custom field definition — if a module happens to define a custom file field with `apiFieldName='standaloneAttachments'`, uploads under that key go to the standalone bucket and are not assigned to that field. Avoid using `standaloneAttachments` as an `apiFieldName` for custom fields.

---

## ✅ File Field Validation

- Files sent under a non-reserved key must match an existing API field name of type `File` in the module schema.
- Files sent under the reserved key `standaloneAttachments` are accepted regardless of the module schema and are stored as standalone attachments.
- Duplicate keys are rejected for named file fields (one file per field). The reserved key `standaloneAttachments` may be repeated to upload multiple standalone files in a single request.
- File size: **max 15MB per file**
- Max files: **10 per request** (named + standalone combined)
- Invalid or extra fields will cause a validation error.

---

## 📋 Response Example

```json
{
  "data": { /* created or updated record object */ },
  "meta": { "deprecation": null }
}
```

---

## 🛠️ Troubleshooting

- **Validation error:**
  Check that all file fields exist in the module and match the API field names.
- **File too large:**
  Ensure each file is under 15MB.
- **Too many files:**
  Limit to 10 files per request.
- **Unsupported field:**
  Only file-type fields are accepted for uploads.
- **Authentication error:**
  Ensure your API key is valid and included in the `x-api-key` header.
- **Rate limit error:**
  Wait and retry if you receive HTTP 429.

---

---

## 📥 Downloading Files

To download files attached to records, use the dedicated file endpoints:

- **List files:** `GET /api/{module}/{recordId}/files` — returns metadata for all files on a record
- **Download file:** `GET /api/files/{fileId}/download` — streams the file binary with proper Content-Disposition headers

See [Endpoints Reference](docs/rest-api/endpoints.md#get-apimodulerecordidfiles) for full documentation.

---

_For more details on modules and field types, see [modules.md](docs/rest-api/modules.md)._