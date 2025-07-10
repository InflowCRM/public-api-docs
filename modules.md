# InflowCRM Public API – Modules & Fields

This document describes all modules supported by the Public API, their field types, requirements, and relationship field usage.

> **Important:** Only fields with an **'Api field name'** set in the InflowCRM UI can be used in the API.
> You can find the "Api field name" for each field in the module settings in the InflowCRM UI.

---

## Supported Modules

- **customer**
- **task**
- **contact**
- **note**
- **order**
- **potential**
- **event**
- **enlist**
- **coach**

Only modules available in your bundle can be accessed.

> **Note:** All supported modules can also receive webhook notifications for create, update, and delete events. See [Webhook API](./webhooks.md) for details.

---

## Module Field Types

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

## Field Requirements

- **required**: Field must be present in `data`
- **readonly**: Field cannot be set via API
- **maxLength/minLength**: For string fields
- **pickListValues**: Allowed values for picklist/multi-pick
- **relatedModule**: For relation fields, specifies the target module

---

## Example: Get Module Fields

Request:
```json
{
  "module": "customer"
}
```

Response:
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

## Relationship Fields

- Use the related module's record ID as the value.
- Example: To set a contact's company, provide the company record's ID.

---

For a full list of fields per module, use the `/api/getModuleFields` endpoint.