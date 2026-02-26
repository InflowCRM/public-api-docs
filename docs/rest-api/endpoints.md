# InflowCRM Public API – Endpoints Reference

This section provides detailed documentation for every public API endpoint, including parameters, request/response examples, and module-specific notes.

---

## 📑 Table of Contents

### Core API Endpoints
- [GET /api/getBundleModuleNames](#get-apigetbundlemodulenames)
- [GET /api/getUserList](#get-apigetuserlist)
- [GET /api/getUser/{id}](#get-apigetuserid)
- [PATCH /api/updateUser/{id}](#patch-apiupdateuserid)
- [POST /api/getProcessesForModule](#post-apigetprocessesformodule)
- [POST /api/validateApiKey](#post-apivalidateapikey)
- [POST /api/createRecord](#post-apicreaterecord)
- [PATCH /api/updateRecord/{id}](#patch-apiupdaterecordid)
- [POST /api/searchRecords](#post-apisearchrecords)
- [POST /api/findRecord](#post-apifindrecord)
- [GET /api/{module}/{id}](#get-apimoduleid)
- [POST /api/records/getByIds](#post-apirecordsgetbyids)
- [POST /api/getModuleFields](#post-apigetmodulefields)
- [POST /api/getSampleData](#post-apigetsampledata)
- [POST /api/records/count](#post-apirecordscount)
- [DELETE /api/{module}/{id}](#delete-apimoduleid)

### File Endpoints
- [GET /api/{module}/{recordId}/files](#get-apimodulerecordidfiles)
- [GET /api/files/{fileId}/download](#get-apifilesfileiddownload)

### Utility Endpoints
- [POST /api/make/getModuleFields](#post-apimakegetmodulefields)

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

## GET /api/getUser/{id}

Retrieves a single user by their ID.

**Headers:**
`x-api-key: YOUR_API_KEY`

**Path Parameters:**
- `id` (string, required): User ID (MongoDB ObjectId)

**Response (200 OK):**
```json
{
  "data": {
    "id": "507f1f77bcf86cd799439011",
    "systemLabel": "Jan Kowalski",
    "email": "jan@example.com",
    "phone": "+48123456789",
    "position": "Sales Manager"
  },
  "meta": { "deprecation": null }
}
```

> **Note:** The response includes all user fields that have an API field name configured. System fields like password, apiKey, and security settings are never exposed.

**Error Responses:**
- **400 Bad Request**: Invalid user ID format
- **404 Not Found**: User not found
- **401 Unauthorized**: Missing or invalid API key

**Example (curl):**
```bash
curl -X GET "https://srv.inflowcrm.pl/api/getUser/507f1f77bcf86cd799439011" \
  -H "x-api-key: YOUR_API_KEY"
```

---

## PATCH /api/updateUser/{id}

Updates custom fields on a user profile.

**Headers:**
`x-api-key: YOUR_API_KEY`
`Content-Type: application/json`

**Path Parameters:**
- `id` (string, required): User ID (MongoDB ObjectId)

**Body:**
```json
{
  "phone": "+48987654321",
  "position": "Senior Sales Manager"
}
```

> **Note:** Only custom fields with an API field name can be updated. System and security fields (e.g., email, password, role) cannot be modified via this endpoint.

**Response (200 OK):**
```json
{
  "data": {
    "id": "507f1f77bcf86cd799439011",
    "systemLabel": "Jan Kowalski",
    "email": "jan@example.com",
    "phone": "+48987654321",
    "position": "Senior Sales Manager"
  },
  "meta": { "deprecation": null }
}
```

**Error Responses:**
- **400 Bad Request**: Empty body or no updatable fields provided
- **404 Not Found**: User not found
- **401 Unauthorized**: Missing or invalid API key

**Example (curl):**
```bash
curl -X PATCH "https://srv.inflowcrm.pl/api/updateUser/507f1f77bcf86cd799439011" \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "phone": "+48987654321" }'
```

---

## POST /api/getProcessesForModule

Returns all available processes for a specific module.

**Headers:**  
`x-api-key: YOUR_API_KEY`

**Body:**
```json
{
  "module": "order"
}
```

> **Note:** This endpoint currently supports the `order` and `potential` modules only.
> The stage index (position in `stagesNames` array, 0-based) is used when setting a record's stage. See [Pipelines & Stages](docs/rest-api/modules.md#pipelines-amp-stages) for details.

**Response:**
```json
{
	"data": [
		{
			"id": "648be3f299a153207a257751",
			"systemLabel": "B2B Sales Process",
			"stagesNames": [
				"New Lead",
				"First Contact",
				"Needs Analysis",
				"Offer Sent",
				"Negotiations",
        "Closed"
			]
		},
		{
			"id": "64b55a77477c55373394a477",
			"systemLabel": "Customer Support Process",
			"stagesNames": [
				"New Ticket",
				"In Progress",
				"Waiting for Customer",
				"Resolved"
			]
		}
	],
	"meta": {
		"deprecation": null
	}
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
  -F "profilePicture=@path/to/photo.jpg"
```

**Response:**
```json
{
  "data": {
    "id": "507f1f77bcf86cd799439011",
    "name": "John Doe",
    "email": "john@example.com",
    "createdAt": "2025-01-15T10:30:00.000Z",
    "modifiedAt": "2025-01-15T10:30:00.000Z"
  },
  "meta": { "deprecation": null }
}
```

> **Note:** The response fields depend on your module configuration. Only fields with an API field name will be included.

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
  -F "profilePicture=@path/to/photo.jpg"
```

**Response:**
```json
{
  "data": {
    "id": "507f1f77bcf86cd799439011",
    "name": "Jane Doe",
    "email": "john@example.com",
    "createdAt": "2025-01-15T10:30:00.000Z",
    "modifiedAt": "2025-01-16T14:22:00.000Z"
  },
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
| `isEmpty`        | Is null, empty string, or empty array       | `true`                 | all                       |
| `isNotEmpty`     | Is not null, empty string, or empty array   | `true`                 | all                       |

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

**Example (isEmpty / isNotEmpty):**
To find all customers that have an assigned contact:
```json
{
  "module": "customer",
  "advancedFilters": [
    {
      "conditions": [
        { "field": "customerContact", "condition": "isNotEmpty", "value": true }
      ]
    }
  ]
}
```
To find all customers that do **not** have an assigned contact, you can use `isEmpty` or negate `isNotEmpty`:
```json
{
  "module": "customer",
  "advancedFilters": [
    {
      "conditions": [
        { "field": "customerContact", "condition": "isEmpty", "value": true }
      ]
    }
  ]
}
```
or
```json
{
  "module": "customer",
  "advancedFilters": [
    {
      "conditions": [
        { "field": "customerContact", "condition": "isNotEmpty", "value": false }
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
| string     | equal, notEqual, in, notIn, isEmpty, isNotEmpty |
| number     | equal, notEqual, greater, greaterOrEqual, lower, lowerOrEqual, in, notIn, isEmpty, isNotEmpty |
| date       | equal, notEqual, greater, greaterOrEqual, lower, lowerOrEqual, isEmpty, isNotEmpty |
| boolean    | equal, notEqual, isEmpty, isNotEmpty |
| relation   | equal, notEqual, in, notIn, isEmpty, isNotEmpty |
| array      | isEmpty, isNotEmpty                  |

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
  "data": {
    "id": "507f1f77bcf86cd799439011",
    "name": "John Doe",
    "email": "john@example.com",
    "status": "active",
    "createdAt": "2025-01-15T10:30:00.000Z",
    "modifiedAt": "2025-01-15T10:30:00.000Z"
  },
  "meta": { "deprecation": null }
}
```

---

## GET /api/{module}/{id}

Retrieves a single record from a specified module by its ID.

**Headers:**
`x-api-key: YOUR_API_KEY`

**Path Parameters:**
- `module` (string, required): Module name (e.g., "customer", "contact", "order") or its translation
- `id` (string, required): Unique record identifier (MongoDB ObjectId)

**Response (200 OK):**
```json
{
  "data": {
    "id": "507f1f77bcf86cd799439011",
    "name": "Jan Kowalski",
    "email": "jan@example.com"
  },
  "meta": {
    "deprecation": null
  }
}
```

**Error Responses:**
- **404 Not Found**: Record not found or has been deleted
- **401 Unauthorized**: Missing or invalid API key
- **403 Forbidden**: Module not available in your bundle

**Example (curl):**
```bash
curl -X GET "https://srv.inflowcrm.pl/api/customer/507f1f77bcf86cd799439011" \
  -H "x-api-key: YOUR_API_KEY"
```

**Notes:**
- Automatically filters out deleted records (`deleted: true`)
- Returns only records belonging to the same tenant (master)
- Fields are mapped from internal format to API format

---

## POST /api/records/getByIds

Retrieves multiple records from a specified module by a list of IDs. Supports pagination for large datasets.

**Headers:**
`x-api-key: YOUR_API_KEY`
`Content-Type: application/json`

**Body:**
```json
{
  "module": "customer",
  "ids": [
    "507f1f77bcf86cd799439011",
    "507f1f77bcf86cd799439012",
    "507f1f77bcf86cd799439013"
  ],
  "page": 1,
  "limit": 50,
  "sort": {
    "createdAt": -1
  }
}
```

**Body Parameters:**
- `module` (string, required): Module name or its translation
- `ids` (array of strings, required): Array of record IDs to retrieve
- `page` (number, optional, default: 1): Page number (min: 1)
- `limit` (number, optional, default: 50): Records per page (min: 1, max: 200)
- `sort` (object, optional): Sort criteria, where `1` = ascending, `-1` = descending

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "507f1f77bcf86cd799439011",
      "name": "Jan Kowalski",
      "email": "jan@example.com"
    },
    {
      "id": "507f1f77bcf86cd799439012",
      "name": "Anna Nowak",
      "email": "anna@example.com"
    }
  ],
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 50,
      "total": 2,
      "pages": 1
    },
    "deprecation": null
  }
}
```

**Error Responses:**
- **400 Bad Request**: Invalid parameters (e.g., missing module, empty ids array)
- **401 Unauthorized**: Missing or invalid API key
- **403 Forbidden**: Module not available in your bundle

**Example (curl):**
```bash
curl -X POST "https://srv.inflowcrm.pl/api/records/getByIds" \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "module": "customer",
    "ids": ["507f1f77bcf86cd799439011", "507f1f77bcf86cd799439012"],
    "page": 1,
    "limit": 50,
    "sort": {"createdAt": -1}
  }'
```

**Notes:**
- Automatically skips non-existent or deleted records
- Returns only records belonging to the same tenant (master)
- Pagination allows efficient retrieval of large record sets
- Maximum limit per page is 200 records
- Default sort is `{ createdAt: -1 }` if not specified

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
  "data": {
    "id": "507f1f77bcf86cd799439011",
    "name": "Example Customer",
    "email": "customer@example.com",
    "status": "active",
    "createdAt": "2025-01-10T08:00:00.000Z",
    "modifiedAt": "2025-01-12T16:45:00.000Z"
  },
  "meta": { "deprecation": null }
}
```

> **Note:** The fields returned depend on the module's configuration. This endpoint returns one sample record to help you discover the available field structure.

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

## GET /api/{module}/{recordId}/files

Lists all files attached to a specific record.

**Headers:**
`x-api-key: YOUR_API_KEY`

**Path Parameters:**
- `module` (string, required): Module name (e.g., "customer", "order")
- `recordId` (string, required): Record ID (MongoDB ObjectId)

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "507f1f77bcf86cd799439011",
      "fileName": "contract.pdf",
      "fileSize": 204800,
      "mimeType": "application/pdf",
      "fileExtension": "pdf",
      "createdTime": "2025-01-15T10:00:00.000Z"
    },
    {
      "id": "507f1f77bcf86cd799439012",
      "fileName": "photo.jpg",
      "fileSize": 1048576,
      "mimeType": "image/jpeg",
      "fileExtension": "jpg",
      "createdTime": "2025-01-16T14:30:00.000Z"
    }
  ],
  "meta": { "deprecation": null }
}
```

**Error Responses:**
- **400 Bad Request**: Invalid ObjectId format
- **404 Not Found**: Record not found
- **401 Unauthorized**: Missing or invalid API key

**Example (curl):**
```bash
curl -X GET "https://srv.inflowcrm.pl/api/customer/507f1f77bcf86cd799439011/files" \
  -H "x-api-key: YOUR_API_KEY"
```

---

## GET /api/files/{fileId}/download

Downloads a file by its ID as a binary stream.

**Headers:**
`x-api-key: YOUR_API_KEY`

**Path Parameters:**
- `fileId` (string, required): File ID (MongoDB ObjectId)

**Response (200 OK):** Binary file stream

**Response Headers:**
```
Content-Type: application/pdf
Content-Disposition: attachment; filename*=UTF-8''contract.pdf
Content-Length: 204800
X-Content-Type-Options: nosniff
Cache-Control: no-cache, no-store, must-revalidate
```

**Error Responses:**
- **400 Bad Request**: Invalid ObjectId format
- **404 Not Found**: File not found or access denied
- **401 Unauthorized**: Missing or invalid API key

> **Note:** Returns 404 (not 403) when the file exists but the user has no access — this prevents information leakage about file existence.

**Example (curl):**
```bash
curl -X GET "https://srv.inflowcrm.pl/api/files/507f1f77bcf86cd799439011/download" \
  -H "x-api-key: YOUR_API_KEY" \
  -o downloaded_file.pdf
```

---

## POST /api/make/getModuleFields

Returns field metadata for a module in [Make.com](https://www.make.com/) compatible format. This endpoint is used by the Make.com integration for dynamic field discovery.

**Headers:**
`x-api-key: YOUR_API_KEY`
`Content-Type: application/json`

**Body:**
```json
{
  "module": "customer",
  "requestType": "createRecord"
}
```

**Parameters:**
- `module` (string, required): Module name
- `requestType` (string, required): Operation context — affects `required` flags. Typically `"createRecord"` or `"updateRecord"`

**Response (200 OK):**
```json
{
  "data": [
    {
      "name": "companyName",
      "label": "Nazwa firmy",
      "type": "text",
      "required": true
    },
    {
      "name": "owner",
      "label": "Właściciel",
      "type": "select",
      "required": false,
      "options": [
        { "value": "507f1f77bcf86cd799439011", "label": "jan@example.com" },
        { "value": "507f1f77bcf86cd799439012", "label": "anna@example.com" }
      ]
    },
    {
      "name": "isActive",
      "label": "Aktywny",
      "type": "boolean",
      "required": false
    }
  ]
}
```

**Type mapping:**
| CRM Field Type | Make.com Type |
|---------------|---------------|
| String, Phone, Email | `text` |
| Number | `number` |
| Date | `date` |
| CheckBox | `boolean` |
| Relation (single) | `select` |
| PickList | `select` (with options) |

> **Note:** MultiRelation fields are excluded from the response. User relation fields populate `options` with user ID/email pairs. Fields are marked `required: true` only for `createRecord` operations.

> **Note:** This endpoint returns a flat `data` array without the standard `meta` wrapper used by other endpoints.

**Error Responses:**
- **400 Bad Request**: Missing module parameter
- **401 Unauthorized**: Missing or invalid API key

**Example (curl):**
```bash
curl -X POST "https://srv.inflowcrm.pl/api/make/getModuleFields" \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "module": "customer", "requestType": "createRecord" }'
```

---

## Module-Specific Notes

- Only modules available in your bundle can be accessed.
- **Only fields with an 'Api field name' set in the InflowCRM UI can be used in the API.**
- Field requirements and types vary by module. See [modules.md](docs/rest-api/modules.md) for details.
- File uploads are only supported on modules with file-type fields.

> For more details on file uploads, see [file-uploads.md](docs/rest-api/file-uploads.md).

---