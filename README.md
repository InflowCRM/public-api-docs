<p align="center">
  <img src="https://raw.githubusercontent.com/arqo/inflowcrm-public-api-docs/main/logo.png" alt="InflowCRM Logo" width="120" style="border-radius:12px;box-shadow:0 2px 8px rgba(0,0,0,0.08);margin-bottom:1rem;">
</p>

<h1 align="center">InflowCRM Public API Documentation</h1>

<p align="center">
  <b>Comprehensive, modern, and easy-to-navigate documentation for the InflowCRM Public API.</b>
</p>

---

Welcome! This site provides everything you need to securely integrate with InflowCRM, automate workflows, and manage CRM data programmatically.

> **Tip:** Use the sidebar to explore endpoints, guides, and references.

---

### 🚀 Get Started

1. **Get your API Key** from your InflowCRM account admin.
2. **Set the base URL:**
   `https://srv.inflowcrm.pl`
3. **Authenticate every request** by adding your API key to the `x-api-key` header.
4. **Note:** Only fields with an **'Api field name'** set in the InflowCRM UI can be used in the API.

**Example (using `curl`):**

## Quick Start

1. **Get your API Key** from your InflowCRM account admin.
2. **Set the base URL**:
   `https://srv.inflowcrm.pl`
3. **Authenticate every request** by adding your API key to the `x-api-key` header.
4. **Important:** Only fields with an **'Api field name'** set in the InflowCRM UI can be used in the API.

```bash
curl -H "x-api-key: YOUR_API_KEY" https://srv.inflowcrm.pl/api/getBundleModuleNames
```

---

## Authentication

All endpoints require an API key via the `x-api-key` header.  
If the key is missing or invalid, a `401 Unauthorized` error is returned.

---

## Base URL

```
https://srv.inflowcrm.pl
```

---

## Rate Limits

- **300 requests per 5-minute window** per (API key + IP)
- Exceeding this limit returns HTTP 429 with a rate limit error

---

## Endpoints Overview

> **Important:** Only fields with an **'Api field name'** set in the InflowCRM UI can be used in the API.

---

## Key Features

- Simple and advanced filtering for all search/count/find endpoints
- Comprehensive support for filter conditions: `equal`, `notEqual`, `greater`, `greaterOrEqual`, `lower`, `lowerOrEqual`, `in`, `notIn`
- Migration path from simple to advanced filters (backward compatible)
- Error handling and field compatibility matrix
- Performance best practices for filtering
- **Real-time webhooks** for create, update, and delete events
- **HMAC-SHA256 signature verification** for webhook security

---

### Core API Endpoints

| Endpoint                        | Method | Description                              |
|----------------------------------|--------|------------------------------------------|
| `/api/getBundleModuleNames`      | GET    | List available modules                   |
| `/api/getUserList`               | GET    | List all active users                    |
| `/api/validateApiKey`            | POST   | Validate API key and get user info       |
| `/api/createRecord`              | POST   | Create a new record (supports files)     |
| `/api/updateRecord/{id}`         | PATCH  | Update a record (supports files)         |
| `/api/searchRecords`             | POST   | Search records with filters/pagination   |
| `/api/findRecord`                | POST   | Find a single record by filters          |
| `/api/getModuleFields`           | POST   | Get metadata for module fields           |
| `/api/getSampleData`             | POST   | Get sample data for a module             |
| `/api/records/count`             | POST   | Count records matching filters           |
| `/api/{module}/{id}`             | DELETE | Delete (soft delete) a record            |

### Webhook Endpoints

| Endpoint                        | Method | Description                              |
|----------------------------------|--------|------------------------------------------|
| `/webhooks/subscribe`           | POST   | Create webhook subscription              |
| `/webhooks`                     | GET    | List webhook subscriptions               |
| `/webhooks/{id}`                | GET    | Get webhook subscription details         |
| `/webhooks/{id}`                | DELETE | Delete webhook subscription              |

> **Note:** All `/integration/*` endpoints are deprecated. Use `/api/*` endpoints for new integrations.

---

## Common Response Format

All successful responses:

```json
{
  "data": ...,
  "meta": {
    "pagination": { "page": 1, "limit": 50, "total": 100, "pages": 2 },
    "deprecation": null
  }
}
```

All error responses:

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

## 📚 Further Reading

- [Endpoints Reference](./endpoints.md)
- [File Upload Guide](./file-uploads.md)
- [Modules & Fields](./modules.md)
- [Webhook API](./webhooks.md)
- [Error Handling](./errors.md)

---

## 📄 OpenAPI Specification

- [Full OpenAPI 3.0 Spec](./openapi.yaml)

---

<p align="center">
  <sub>© 2025 InflowCRM &middot; <a href="https://github.com/arqo/inflowcrm-public-api-docs">View on GitHub</a></sub>
</p>