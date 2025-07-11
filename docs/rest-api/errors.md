# InflowCRM Public API – Error Handling

This document explains error responses, status codes, and common error scenarios for the Public API.

---

## 📦 Error Response Structure

All errors use a consistent format:

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable error message",
    "details": {}
  }
}
```

---

## 🏷️ HTTP Status Codes

| Code | Meaning                  | When Returned                        |
|------|--------------------------|--------------------------------------|
| 400  | Bad Request              | Validation failed, invalid input     |
| 401  | Unauthorized             | Missing or invalid API key           |
| 404  | Not Found                | Resource does not exist              |
| 409  | Conflict                 | Duplicate or conflicting data        |
| 415  | Unsupported Media Type   | Wrong content type                   |
| 422  | Unprocessable Entity     | Semantic validation error            |
| 429  | Too Many Requests        | Rate limit exceeded                  |
| 500  | Internal Server Error    | Unexpected server error              |

---

## 🗂️ Common Error Codes

| Code                  | Description                                  |
|-----------------------|----------------------------------------------|
| VALIDATION_ERROR      | Request body or parameters failed validation |
| UNAUTHORIZED          | API key missing or invalid                   |
| NOT_FOUND             | Resource not found                           |
| RATE_LIMIT_EXCEEDED   | Too many requests (see below)                |
| INTERNAL_SERVER_ERROR | Unexpected server error                      |
| FORBIDDEN             | Access to module or resource denied          |

---

## 🚦 Rate Limiting

- **Limit:** 300 requests per 5-minute window per (API key + IP)
- **Error:** HTTP 429 with code `RATE_LIMIT_EXCEEDED`
- **Response Example:**
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Please try again later.",
    "details": null
  }
}
```
- Wait before retrying if you receive this error.

---

## ⚠️ Deprecation Warnings

Deprecated `/integration/*` endpoints return a deprecation warning in the `meta.deprecation` field.

---

## 🛠️ Troubleshooting

- **Validation errors:** Check request body and field types.
  > **Note:** Only fields with an **'Api field name'** set in the InflowCRM UI can be used in the API.
- **Authentication errors:** Ensure your API key is valid and present in the `x-api-key` header.
- **Module access errors:** Only modules in your bundle are accessible.
- **File upload errors:** Check file size, count, and field names.

---

_For more details, see [README.md](./README.md) and [endpoints.md](./endpoints.md)._