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
> **Note:** Sorting can only be performed on a single field at a time.

**Body (Advanced filter):**
```json
{
  "module": "customer",
  "advancedFilters": [
    {
      "conditions": [
        { "field": "amount", "condition": "greaterOrEqual", "value": 100 },
        { "field": "amount", "condition": "lower", "value": 500 },
        { "field": "status", "condition": "in", "value": ["active", "pending"] },
        { "field": "createdAt", "condition": "greater", "value": "2024-01-01" }
      ]
    }
  ],
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
  "advancedFilters": [
    {
      "conditions": [
        { "field": "amount", "condition": "greaterOrEqual", "value": 100 },
        { "field": "amount", "condition": "lower", "value": 500 }
      ]
    },
    {
      "conditions": [
        { "field": "status", "condition": "in", "value": ["active", "pending"] }
      ]
    }
  ]
}
```

**Combining Conditions:**
The `advancedFilters` parameter accepts an array of condition groups.
- All conditions within a single group (inside the `conditions` array) are combined with a logical **AND**.
- Multiple condition groups in the top-level `advancedFilters` array are combined with a logical **OR**.

This allows for building complex queries, such as `(condition1 AND condition2) OR (condition3)`.

**Migration Guide:**
- **Simple:** `{ "filters": { "status": "active" } }`
- **Advanced:** `{ "advancedFilters": [{ "conditions": [{ "field": "status", "condition": "equal", "value": "active" }] }] }`
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
  "advancedFilters": [
    {
      "conditions": [
        { "field": "email", "condition": "equal", "value": "john@example.com" }
      ]
    }
  ]
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
      "apiFieldName": "email",
      "description": "User's email address",
      "decimal": null,
      "length": 255,
      "type": "string",
      "label": "Email",
      "required": true,
      "relationField": null,
      "pickListValues": [],
      "relatedModule": null,
      "relationFieldName": null,
      "defaultValue": null
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
  "advancedFilters": [
    {
      "conditions": [
        { "field": "status", "condition": "notEqual", "value": "archived" },
        { "field": "amount", "condition": "greater", "value": 100 }
      ]
    }
  ]
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

Deletes a record.

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