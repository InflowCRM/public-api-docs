# InflowCRM Public API – Endpoints Reference

This section provides detailed documentation for every public API endpoint, including parameters, request/response examples, and module-specific notes.

---

## 📑 Table of Contents

### Core API Endpoints
- [GET /api/getBundleModuleNames](#get-apigetbundlemodulenames)
- [GET /api/getUserList](#get-apigetuserlist)
- [POST /api/validateApiKey](#post-apivalidateapikey)
- [POST /api/createRecord](#post-apicreaterecord)
- [PATCH /api/updateRecordid](#patch-apiupdaterecordid)
- [POST /api/searchRecords](#post-apisearchrecords)
- [POST /api/findRecord](#post-apifindrecord)
- [POST /api/getModuleFields](#post-apigetmodulefields)
- [POST /api/getSampleData](#post-apigetsampledata)
- [POST /api/records/count](#post-apirecordscount)
- [DELETE /api/{module}/{id}](#delete-apimoduleid)

### Webhook Endpoints
- [POST /webhooks/subscribe](#post-webhookssubscribe)
- [GET /webhooks](#get-webhooks)
- [GET /webhooks/{id}](#get-webhooksid)
- [DELETE /webhooks/{id}](#delete-webhooksid)

---

_Navigate using the sidebar or this table of contents. All endpoints are grouped by function for easier browsing._

## GET /api/getBundleModuleNames

Returns all modules available to the current user’s bundle.

**Headers:**  
`x-api-key: YOUR_API_KEY`

**Response:**
```json
{
  "data": [
    {
      "moduleName": "customer",
      "singular": "Klient",
      "plural": "Klienci",
      "systemModuleName": "customer"
    }
  ],
  "meta": { "deprecation": null }
}
```

---

## GET /api/getUserList

Returns all active users for the current tenant.

**Headers:**  
`x-api-key: YOUR_API_KEY`

**Response:**
```json
{
  "data": [
    {
      "id": "507f1f77bcf86cd799439011",
      "systemLabel": "John Doe",
      "email": "john.doe@example.com"
    }
  ],
  "meta": { "deprecation": null }
}
```

---

## POST /api/validateApiKey

Validates an API key and returns user info.

**Headers:**  
`x-api-key: YOUR_API_KEY`

**Body:**
```json
{
  "apiKey": "YOUR_API_KEY"
}
```

**Response:**
```json
{
  "data": {
    "valid": true,
    "userId": "507f1f77bcf86cd799439011",
    "companyId": "507f1f77bcf86cd799439012",
    "userType": "admin"
  },
  "meta": { "deprecation": null }
}
```

---

## POST /api/createRecord

Creates a new record in a module. Supports file upload via `multipart/form-data`.

> **Important:** All fields must be sent as top-level form fields in multipart requests. **Do not nest fields under `data`.**
> **Only fields with an 'Api field name' set in the InflowCRM UI can be used in the API.**

**Headers:**
`x-api-key: YOUR_API_KEY`

**Body (JSON):**
```json
{
  "module": "customer",
  "data": {
    "name": "John Doe",
    "email": "john@example.com"
  }
}
```

**Body (multipart/form-data):**
- `module`: customer
- `name`: John Doe
- `email`: john@example.com
- `fileFieldName`: (file)

**Example (curl):**
```bash
curl -X POST "https://srv.inflowcrm.pl/api/createRecord" \
  -H "x-api-key: YOUR_API_KEY" \
  -F "module=customer" \
  -F "name=John Doe" \
  -F "email=john@example.com" \
  -F "profilePicture=@/path/to/photo.jpg"
```

**Response:**
```json
{
  "data": { /* created record object */ },
  "meta": { "deprecation": null }
}
```

---

## PATCH /api/updateRecord/{id}

Updates an existing record. Supports file upload via `multipart/form-data`.

> **Important:** All fields must be sent as top-level form fields in multipart requests. **Do not nest fields under `data`.**
> **Only fields with an 'Api field name' set in the InflowCRM UI can be used in the API.**

**Headers:**
`x-api-key: YOUR_API_KEY`

**Path Parameters:**
- `id`: Record ID

**Body (JSON):**
```json
{
  "module": "customer",
  "data": {
    "name": "Jane Doe"
  }
}
```

**Body (multipart/form-data):**
- `module`: customer
- `name`: Jane Doe
- `fileFieldName`: (file)

**Example (curl):**
```bash
curl -X PATCH "https://srv.inflowcrm.pl/api/updateRecord/RECORD_ID" \
  -H "x-api-key: YOUR_API_KEY" \
  -F "module=customer" \
  -F "name=Jane Doe" \
  -F "profilePicture=@/path/to/photo.jpg"
```

**Response:**
```json
{
  "data": { /* updated record object */ },
  "meta": { "deprecation": null }
}
```

---

## POST /api/searchRecords

Searches records with filters and pagination.

**Headers:**
`x-api-key: YOUR_API_KEY`

**Body (Simple filter):**
```json
{
  "module": "customer",
  "filters": { "status": "active" },
  "page": 1,
  "limit": 50,
  "sort": { "createdAt": -1 }
}
```

**Body (Advanced filter):**
```json
{
  "module": "customer",
  "advancedFilters": {
    "amount": [
      { "condition": "greaterOrEqual", "value": 100 },
      { "condition": "lower", "value": 500 }
    ],
    "status": { "condition": "in", "value": ["active", "pending"] },
    "createdAt": { "condition": "greater", "value": "2024-01-01" }
  },
  "page": 1,
  "limit": 50
}
```

**Response:**
```json
{
  "data": [ /* array of records */ ],
  "meta": {
    "pagination": { "page": 1, "limit": 50, "total": 100, "pages": 2 },
    "deprecation": null
  }
}
```

---

### Advanced Filtering

The `advancedFilters` property enables complex queries using supported conditions:

| Condition        | Description                                 | Example Value           | Field Types Supported      |
|------------------|---------------------------------------------|------------------------|---------------------------|
| `equal`          | Equals                                      | `"active"`             | string, number, date      |
| `notEqual`       | Not equals                                  | `"archived"`           | string, number, date      |
| `greater`        | Greater than                                | `100`                  | number, date              |
| `greaterOrEqual` | Greater than or equal                       | `100`                  | number, date              |
| `lower`          | Less than                                   | `500`                  | number, date              |
| `lowerOrEqual`   | Less than or equal                          | `500`                  | number, date              |
| `in`             | In array                                    | `[ "active", "pending" ]` | string, number         |
| `notIn`          | Not in array                                | `[ "archived" ]`       | string, number            |

**Example:**
```json
{
  "advancedFilters": {
    "amount": [
      { "condition": "greaterOrEqual", "value": 100 },
      { "condition": "lower", "value": 500 }
    ],
    "status": { "condition": "in", "value": ["active", "pending"] }
  }
}
```

**Combining Conditions:**
Multiple conditions for a field are combined with logical AND.
To filter by OR across fields, use multiple fields in `advancedFilters`.

**Migration Guide:**
- **Simple:** `{ "filters": { "status": "active" } }`
- **Advanced:** `{ "advancedFilters": { "status": { "condition": "equal", "value": "active" } } }`
- Both can be used for backward compatibility.

**Error Handling:**
- Invalid condition or value type returns a validation error.
- Unsupported field/condition combination returns a descriptive error.

**Performance & Best Practices:**
- Use indexed fields for best performance.
- Avoid excessive use of `in`/`notIn` with large arrays.
- Prefer simple filters when possible for legacy compatibility.

**Field Compatibility Matrix:**

| Field Type | Supported Conditions                  |
|------------|--------------------------------------|
| string     | equal, notEqual, in, notIn           |
| number     | equal, notEqual, greater, greaterOrEqual, lower, lowerOrEqual, in, notIn |
| date       | equal, notEqual, greater, greaterOrEqual, lower, lowerOrEqual |
| boolean    | equal, notEqual                      |

---


## POST /api/findRecord

Finds a single record by filters.

**Headers:**
`x-api-key: YOUR_API_KEY`

**Body (Simple filter):**
```json
{
  "module": "customer",
  "filters": { "email": "john@example.com" }
}
```

**Body (Advanced filter):**
```json
{
  "module": "customer",
  "advancedFilters": {
    "email": { "condition": "equal", "value": "john@example.com" }
  }
}
```

**Response:**
```json
{
  "data": { /* record object */ },
  "meta": { "deprecation": null }
}
```

---

## POST /api/getModuleFields

Returns metadata for all fields in a module.

**Headers:**  
`x-api-key: YOUR_API_KEY`

**Body:**
```json
{
  "module": "customer"
}
```

**Response:**
```json
{
  "data": [
    {
      "fieldId": "cf1",
      "apiFieldName": "email",
      "type": "string",
      "label": "Email",
      "required": true
    }
  ],
  "meta": { "deprecation": null }
}
```

---

## POST /api/getSampleData

Returns sample data for a module.

**Headers:**  
`x-api-key: YOUR_API_KEY`

**Body:**
```json
{
  "module": "customer"
}
```

**Response:**
```json
{
  "data": { /* sample record object */ },
  "meta": { "deprecation": null }
}
```

---

## POST /api/records/count

Counts records matching filters.

**Headers:**
`x-api-key: YOUR_API_KEY`

**Body (Simple filter):**
```json
{
  "module": "customer",
  "filters": { "status": "active" }
}
```

**Body (Advanced filter):**
```json
{
  "module": "customer",
  "advancedFilters": {
    "status": { "condition": "notEqual", "value": "archived" },
    "amount": { "condition": "greater", "value": 100 }
  }
}
```

**Response:**
```json
{
  "data": { "count": 42 },
  "meta": { "deprecation": null }
}
```

---

## DELETE /api/{module}/{id}

Deletes (soft delete) a record.

**Headers:**  
`x-api-key: YOUR_API_KEY`

**Path Parameters:**
- `module`: Module name
- `id`: Record ID

**Response:**  
HTTP 204 No Content

---

## Module-Specific Notes

- Only modules available in your bundle can be accessed.
- **Only fields with an 'Api field name' set in the InflowCRM UI can be used in the API.**
- Field requirements and types vary by module. See [modules.md](./modules.md) for details.
- File uploads are only supported on modules with file-type fields.

> For more details on file uploads, see [file-uploads.md](./file-uploads.md).

---

## POST /webhooks/subscribe

Creates a new webhook subscription for a specific module and event.

**Headers:**
```
x-api-key: YOUR_API_KEY
Content-Type: application/json
```

**Request Body:**
```json
{
  "event": "create",
  "hookUrl": "https://your-app.com/webhook",
  "module": "customer"
}
```

**Parameters:**
- `event` (string, required): Event type (`create`, `update`, or `delete`)
- `hookUrl` (string, required): Your webhook endpoint URL (HTTPS recommended)
- `module` (string, required): Module name or canonical module identifier

**Response:**
```json
{
  "data": {
    "id": "uuid-subscription-id"
  }
}
```

**Example:**
```bash
curl -X POST "https://srv.inflowcrm.pl/webhooks/subscribe" \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "event": "create",
    "hookUrl": "https://your-app.com/webhook/customer-created",
    "module": "customer"
  }'
```

---

## GET /webhooks

Returns all webhook subscriptions for your account.

**Headers:**
```
x-api-key: YOUR_API_KEY
```

**Query Parameters:**
- `page` (integer, optional): Page number (default: 1)
- `perPage` (integer, optional): Records per page (default: 20, max: 100)

**Response:**
```json
{
  "data": [
    {
      "subscriptionId": "uuid-subscription-id",
      "eventName": "create",
      "hookUrl": "https://your-app.com/webhook",
      "module": "customer",
      "createdAt": "2025-01-07T10:00:00Z",
      "modifiedAt": "2025-01-07T10:00:00Z"
    }
  ]
}
```

**Example:**
```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://srv.inflowcrm.pl/webhooks?page=1&perPage=20"
```

---

## GET /webhooks/{id}

Retrieves details of a specific webhook subscription.

**Headers:**
```
x-api-key: YOUR_API_KEY
```

**Path Parameters:**
- `id` (string, required): Subscription ID

**Response:**
```json
{
  "data": {
    "subscriptionId": "uuid-subscription-id",
    "eventName": "create",
    "hookUrl": "https://your-app.com/webhook",
    "module": "customer",
    "createdAt": "2025-01-07T10:00:00Z",
    "modifiedAt": "2025-01-07T10:00:00Z"
  }
}
```

**Example:**
```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://srv.inflowcrm.pl/webhooks/uuid-subscription-id"
```

---

## DELETE /webhooks/{id}

Removes a webhook subscription.

**Headers:**
```
x-api-key: YOUR_API_KEY
```

**Path Parameters:**
- `id` (string, required): Subscription ID

**Response:**
```json
{
  "data": {
    "deleted": true
  }
}
```

**Example:**
```bash
curl -X DELETE \
  -H "x-api-key: YOUR_API_KEY" \
  "https://srv.inflowcrm.pl/webhooks/uuid-subscription-id"
```

---

## Advanced Filtering – Error Scenarios

- **Invalid Condition:**
  ```json
  {
    "error": {
      "code": "VALIDATION_ERROR",
      "message": "Unsupported filter condition: 'contains'",
      "details": {}
    }
  }
  ```
- **Type Mismatch:**
  ```json
  {
    "error": {
      "code": "VALIDATION_ERROR",
      "message": "Condition 'greater' requires a number or date value.",
      "details": {}
    }
  }
  ```
- **Unsupported Field:**
  ```json
  {
    "error": {
      "code": "VALIDATION_ERROR",
      "message": "Field 'profilePicture' does not support advanced filtering.",
      "details": {}
    }
  }
  ```

---