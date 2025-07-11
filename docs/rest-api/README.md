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
- Comprehensive support for filter conditions: `equal`, `notEqual`, `greater`, `greaterOrEqual`, `lower`, `lowerOrEqual`, `in`, `notIn`
- Migration path from simple to advanced filters (backward compatible)
- Error handling and field compatibility matrix
- Performance best practices for filtering
- **Real-time webhooks** for create, update, and delete events
- **HMAC-SHA256 signature verification** for webhook security

---

## 📚 Documentation Index

- [Endpoints Reference](./endpoints.md)
- [File Upload Guide](./file-uploads.md)
- [Modules & Fields](./modules.md)
- [Webhook API](./webhooks.md)
- [Error Handling](./errors.md)
- [Full OpenAPI 3.0 Spec](./openapi.yaml)

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
- Review the [OpenAPI Spec](./openapi.yaml) for schema details.
- See [Endpoints Reference](./endpoints.md) for all available API methods.
- Check [webhooks.md](./webhooks.md) for real-time integration.

---

<sub align="center">© 2025 InflowCRM &middot; [View on GitHub](https://github.com/arqo/inflowcrm-public-api-docs)</sub>