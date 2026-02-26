# InflowCRM Public API – Modules & Fields

This document describes all modules supported by the Public API, their field types, requirements, and relationship field usage.

> **Important:** Only fields with an **'Api field name'** set in the InflowCRM UI can be used in the API.
> You can find the "Api field name" for each field in the module settings in the InflowCRM UI.

---

## 📦 Supported Modules

| Module | Description |
|--------|-------------|
| `customer` | Companies and clients — the primary CRM entity |
| `contact` | Contact persons associated with companies |
| `order` | Sales orders with pipeline processes |
| `potential` | Sales opportunities and deals in negotiation |
| `task` | Tasks and activities assigned to users |
| `note` | Text notes attached to records |
| `event` | Calendar events and meetings |
| `enlist` | Recruitment entries |
| `coach` | Coaching sessions |
| `room` | Rooms and spaces (e.g. meeting rooms, hotel rooms) |

Only modules available in your bundle can be accessed.

> **Note:** All supported modules can also receive webhook notifications for create, update, and delete events. See [Webhook API](docs/rest-api/webhooks.md) for details.

---

## 📝 Module Field Types

Each module has a set of fields with specific types and requirements.
Common field types:

| Type        | Description                        | Example Value         |
|-------------|------------------------------------|----------------------|
| string      | Text field                         | "John Doe"           |
| number      | Numeric value                      | 42                   |
| boolean     | True/false                         | true                 |
| date        | ISO 8601 date string               | "2025-07-07"         |
| picklist    | One of allowed values              | "active"             |
| multi-pick  | Array of allowed values            | ["a", "b"]           |
| file        | File upload (see file-uploads.md)  | (file)               |
| relation    | Reference to another module record | "507f1f77bcf86cd799439011" |

---

## ✅ Field Requirements

- **required**: Field must be present in `data`
- **readonly**: Field cannot be set via API
- **maxLength/minLength**: For string fields
- **pickListValues**: Allowed values for picklist/multi-pick
- **relatedModule**: For relation fields, specifies the target module

---

## 📋 Example: Get Module Fields

**Request:**
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
      "required": true,
      "readonly": false,
      "maxLength": 100
    },
    {
      "fieldId": "cf2",
      "apiFieldName": "profilePicture",
      "type": "file",
      "label": "Profile Picture",
      "required": false
    },
    {
      "fieldId": "cf3",
      "apiFieldName": "company",
      "type": "relation",
      "label": "Company",
      "relatedModule": "customer"
    }
  ],
  "meta": { "deprecation": null }
}
```

---

## 🔗 Relationship Fields

- Use the related module's record ID as the value.
- Example: To set a contact's company, provide the company record's ID.

---

## 🔄 Pipelines & Stages

The `order` and `potential` modules support **pipelines** (also called processes) — a series of stages that a record moves through (e.g., a sales funnel or support workflow).

### Which modules support pipelines?

| Module | Pipeline support |
|--------|-----------------|
| `order` | Yes |
| `potential` | Yes |
| All other modules | No |

### Getting available pipelines

Use [`POST /api/getProcessesForModule`](docs/rest-api/endpoints.md#post-apigetprocessesformodule) to retrieve all pipelines and their stages:

```json
// Request
{ "module": "order" }

// Response
{
  "data": [
    {
      "id": "648be3f299a153207a257751",
      "systemLabel": "B2B Sales Process",
      "stagesNames": ["New Lead", "First Contact", "Needs Analysis", "Offer Sent", "Negotiations", "Closed"]
    }
  ]
}
```

### Setting pipeline and stage via API

To assign a pipeline and stage when creating or updating a record, you need **two fields** configured with API field names in the InflowCRM UI:

1. **Pipeline field** (relation type) — send the **pipeline ObjectId** from `getProcessesForModule`
2. **Stage field** — send a **0-based numeric index** matching the position in `stagesNames`

**Example:** To place a record in the "Needs Analysis" stage of the pipeline above:

```json
{
  "myPipelineField": "648be3f299a153207a257751",
  "myStageField": 2
}
```

Where `2` corresponds to `stagesNames[2]` = "Needs Analysis" (0 = "New Lead", 1 = "First Contact", 2 = "Needs Analysis", ...).

### Auto-assignment behavior

When creating or updating a record:

- **Pipeline set, stage omitted** — the system automatically assigns **stage 0** (the first stage in the pipeline)
- **Pipeline set, stage provided** — the system uses the provided stage index
- **Pipeline not set** — no pipeline processing occurs

> **Important:** Always send the stage as a **numeric index** (0, 1, 2, ...), not as a stage name string. Sending a stage name will not be converted automatically and will result in the record not appearing correctly on the pipeline board.

### Retrieving pipeline data from records

When you read a record that has a pipeline assigned, the stage and pipeline fields will contain **human-readable names** (not numeric indices). The system converts the stage index to the stage name when saving the record.

---

_For a full list of fields per module, use the `/api/getModuleFields` endpoint._