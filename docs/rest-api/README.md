# InflowCRM Public API Documentation

![InflowCRM Logo](../../logo.png)

---

## Welcome

**InflowCRM Public API** lets you securely integrate, automate, and manage CRM data programmatically.
This documentation is designed for easy navigation and quick onboarding.

> **Tip:** Use the sidebar to browse endpoints, guides, and references.

---

## 🚀 Quick Start

1. **Get your API Key** from your InflowCRM account admin.
2. **Set the base URL:**
   `https://srv.inflowcrm.pl`
3. **Authenticate every request** by adding your API key to the `x-api-key` header.
4. **Note:** Only fields with an **'Api field name'** set in the InflowCRM UI can be used in the API.

**Example (using `curl`):**
```bash
curl -H "x-api-key: YOUR_API_KEY" https://srv.inflowcrm.pl/api/getBundleModuleNames
```

---

## 🔑 Authentication

All endpoints require an API key via the `x-api-key` header.
If the key is missing or invalid, a `401 Unauthorized` error is returned.

---

## 🌐 Base URL

```
https://srv.inflowcrm.pl
```

---

## ⏱️ Rate Limits

- **300 requests per 5-minute window** per (API key + IP)
- Exceeding this limit returns HTTP 429 with a rate limit error

---

## 🧭 API Overview

- Simple and advanced filtering for all search/count/find endpoints
- **Record retrieval** by single ID or multiple IDs with pagination
- Comprehensive support for filter conditions: `equal`, `notEqual`, `greater`, `greaterOrEqual`, `lower`, `lowerOrEqual`, `in`, `notIn`, `isEmpty`, `isNotEmpty`
- Migration path from simple to advanced filters (backward compatible)
- **User management** — get and update user profiles
- **File operations** — list record files and download by ID
- Error handling and field compatibility matrix
- Performance best practices for filtering
- **Real-time webhooks** for create, update, and delete events
- **HMAC-SHA256 signature verification** for webhook security

---

## 📚 Documentation Index

- [Endpoints Reference](docs/rest-api/endpoints.md)
- [File Upload Guide](docs/rest-api/file-uploads.md)
- [Modules & Fields](docs/rest-api/modules.md)
- [Webhook API](docs/rest-api/webhooks.md)
- [Error Handling](docs/rest-api/errors.md)
- [Full OpenAPI 3.0 Spec](docs/rest-api/openapi.yaml)

---

## 📄 Common Response Format

**Success:**
```json
{
  "data": ...,
  "meta": {
    "pagination": { "page": 1, "limit": 50, "total": 100, "pages": 2 },
    "deprecation": null
  }
}
```

**Error:**
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

## 📝 Next Steps

- Explore endpoints and guides using the sidebar.
- Review the [OpenAPI Spec](docs/rest-api/openapi.yaml) for schema details.
- See [Endpoints Reference](docs/rest-api/endpoints.md) for all available API methods.
- Check [webhooks.md](docs/rest-api/webhooks.md) for real-time integration.

---

<sub align="center">© 2026 InflowCRM &middot; [View on GitHub](https://github.com/InflowCRM/public-api-docs)</sub>