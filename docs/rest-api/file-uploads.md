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

## ✅ File Field Validation

- Only fields defined as file-type in the module schema are accepted.
- Each file field must match the API field name.
- Duplicate file keys are rejected.
- File size: **max 15MB per file**
- Max files: **10 per request**
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

_For more details on modules and field types, see [modules.md](./modules.md)._