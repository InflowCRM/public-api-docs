# InflowCRM GraphQL API – Operations Reference

This document provides detailed information about all available GraphQL operations, including queries, mutations, parameters, and examples.

---

## Table of Contents

### Query Operations
- [Module.getOne](#modulegetone)
- [Module.getOneById](#modulegetonebyid)
- [Module.getMany](#modulegetmany)
- [Module.getManyByIds](#modulegetmanybyids)

### Mutation Operations
- [Module.create](#modulecreate)
- [Module.update](#moduleupdate)
- [Module.delete](#moduledelete)

### Common Parameters
- [Filtering](#filtering)
- [Pagination](#pagination)
- [Field Selection](#field-selection)

---

## Query Operations

### Module.getOne

Retrieves a single record based on filtering criteria.

**Arguments:**
- `filtration` - Filter criteria (optional)

**Example:**
```graphql
query {
  Customer {
    getOne(filtration: { fields: { email: { equal: "john@example.com" } } }) {
      id
      createdTime
      modifiedTime
      fields {
        name
        email
        phone
      }
    }
  }
}
```

**Response:**
```json
{
  "data": {
    "Customer": {
      "getOne": {
        "id": "507f1f77bcf86cd799439011",
        "createdTime": "2023-01-15T10:30:00Z",
        "modifiedTime": "2023-01-15T10:30:00Z",
        "fields": {
          "name": "John Doe",
          "email": "john@example.com",
          "phone": "+1234567890"
        }
      }
    }
  }
}
```

---

### Module.getOneById

Retrieves a single record by its ID.

**Arguments:**
- `id` - Record ID (required)

**Example:**
```graphql
query {
  Customer {
    getOneById(id: "507f1f77bcf86cd799439011") {
      id
      createdTime
      fields {
        name
        email
      }
    }
  }
}
```

**Response:**
```json
{
  "data": {
    "Customer": {
      "getOneById": {
        "id": "507f1f77bcf86cd799439011",
        "createdTime": "2023-01-15T10:30:00Z",
        "fields": {
          "name": "John Doe",
          "email": "john@example.com"
        }
      }
    }
  }
}
```

---

### Module.getMany

Retrieves multiple records with filtering and pagination.

**Arguments:**
- `perPage` - Number of items per page (required, default: 20, max: 100)
- `page` - Page number (required, default: 1)
- `filtration` - Filter criteria (optional)

**Example:**
```graphql
query {
  Customer {
    getMany(
      perPage: 10
      page: 1
      filtration: { 
        fields: { 
          name: { contains: "John" } 
        } 
      }
    ) {
      id
      createdTime
      fields {
        name
        email
        status
      }
    }
  }
}
```

**Response:**
```json
{
  "data": {
    "Customer": {
      "getMany": [
        {
          "id": "507f1f77bcf86cd799439011",
          "createdTime": "2023-01-15T10:30:00Z",
          "fields": {
            "name": "John Doe",
            "email": "john@example.com",
            "status": "Active"
          }
        },
        {
          "id": "507f1f77bcf86cd799439012",
          "createdTime": "2023-01-16T11:15:00Z",
          "fields": {
            "name": "John Smith",
            "email": "john.smith@example.com",
            "status": "Inactive"
          }
        }
      ]
    }
  }
}
```

---

### Module.getManyByIds

Retrieves multiple records by their IDs.

**Arguments:**
- `perPage` - Number of items per page (required, default: 20)
- `page` - Page number (required, default: 1)
- `ids` - Array of record IDs (required)

**Example:**
```graphql
query {
  Customer {
    getManyByIds(
      perPage: 10
      page: 1
      ids: ["507f1f77bcf86cd799439011", "507f1f77bcf86cd799439012"]
    ) {
      id
      fields {
        name
        email
      }
    }
  }
}
```

**Response:**
```json
{
  "data": {
    "Customer": {
      "getManyByIds": [
        {
          "id": "507f1f77bcf86cd799439011",
          "fields": {
            "name": "John Doe",
            "email": "john@example.com"
          }
        },
        {
          "id": "507f1f77bcf86cd799439012",
          "fields": {
            "name": "Jane Smith",
            "email": "jane@example.com"
          }
        }
      ]
    }
  }
}
```

---

## Mutation Operations

### Module.create

Creates a new record in the specified module.

**Arguments:**
- `data` - Record data (required)
  - `fields` - Custom field values
  - `relationFields` - Related record data (optional)

**Example:**
```graphql
mutation {
  Customer {
    create(data: {
      fields: {
        name: "Alice Johnson"
        email: "alice@example.com"
        phone: "+1987654321"
      }
    }) {
      ... on Customer {
        id
        createdTime
        fields {
          name
          email
          phone
        }
      }
      ... on ValidationError {
        validationErrors {
          fieldName
          errorType
          description
          systemLabel
        }
      }
    }
  }
}
```

**Success Response:**
```json
{
  "data": {
    "Customer": {
      "create": {
        "id": "507f1f77bcf86cd799439013",
        "createdTime": "2023-01-17T09:45:00Z",
        "fields": {
          "name": "Alice Johnson",
          "email": "alice@example.com",
          "phone": "+1987654321"
        }
      }
    }
  }
}
```

**Validation Error Response:**
```json
{
  "data": {
    "Customer": {
      "create": {
        "validationErrors": [
          {
            "fieldName": "email",
            "errorType": "REQUIRED",
            "description": "Email is required",
            "systemLabel": "Email"
          }
        ]
      }
    }
  }
}
```

---

### Module.update

Updates an existing record.

**Arguments:**
- `id` - Record ID (required)
- `data` - Updated record data (required)
  - `fields` - Custom field values
  - `relationFields` - Related record data (optional)

**Example:**
```graphql
mutation {
  Customer {
    update(
      id: "507f1f77bcf86cd799439011"
      data: {
        fields: {
          name: "John Doe Updated"
          status: "Active"
        }
      }
    ) {
      ... on Customer {
        id
        modifiedTime
        fields {
          name
          email
          status
        }
      }
      ... on ValidationError {
        validationErrors {
          fieldName
          errorType
          description
        }
      }
    }
  }
}
```

**Success Response:**
```json
{
  "data": {
    "Customer": {
      "update": {
        "id": "507f1f77bcf86cd799439011",
        "modifiedTime": "2023-01-17T10:30:00Z",
        "fields": {
          "name": "John Doe Updated",
          "email": "john@example.com",
          "status": "Active"
        }
      }
    }
  }
}
```

---

### Module.delete

Deletes a record.

**Arguments:**
- `id` - Record ID (required)

**Example:**
```graphql
mutation {
  Customer {
    delete(id: "507f1f77bcf86cd799439011") {
      result
    }
  }
}
```

**Response:**
```json
{
  "data": {
    "Customer": {
      "delete": {
        "result": true
      }
    }
  }
}
```

---

## Common Parameters

### Filtering

The `filtration` parameter supports advanced filtering with multiple comparison operators:

```graphql
filtration: {
  fields: {
    fieldName: {
      equal: "value"
      notEqual: "value"
      contains: "substring"
      notContains: "substring"
      in: ["value1", "value2"]
      notIn: ["value1", "value2"]
      greater: 100
      greaterOrEqual: 100
      lower: 100
      lowerOrEqual: 100
      isNull: true
      isNotNull: true
    }
  }
  systemFields: {
    id: { equal: "507f1f77bcf86cd799439011" }
    createdTime: { greater: "2023-01-01T00:00:00Z" }
    modifiedTime: { lower: "2023-12-31T23:59:59Z" }
  }
}
```

**Available Operators:**
- `equal` - Exact match
- `notEqual` - Not equal
- `contains` - Contains substring (case-insensitive)
- `notContains` - Does not contain substring
- `in` - Value is in array
- `notIn` - Value is not in array
- `greater` - Greater than (numbers/dates)
- `greaterOrEqual` - Greater than or equal
- `lower` - Less than (numbers/dates)
- `lowerOrEqual` - Less than or equal
- `isNull` - Field is null/empty
- `isNotNull` - Field is not null/empty

### Pagination

All `getMany` and `getManyByIds` operations support pagination:

```graphql
{
  perPage: 20    # Items per page (max: 100)
  page: 1        # Page number (starts from 1)
}
```

### Field Selection

GraphQL allows you to request only the fields you need:

```graphql
query {
  Customer {
    getMany(perPage: 10, page: 1) {
      id                    # System field
      createdTime          # System field
      modifiedTime         # System field
      fields {             # Custom fields
        name
        email
        # Only request needed fields
      }
      # Relational fields
      contacts {
        id
        fields {
          firstName
          lastName
        }
      }
    }
  }
}
```

---

## Relational Queries

Fetch related data in a single request:

```graphql
query {
  Customer {
    getOne(filtration: { fields: { email: { equal: "john@example.com" } } }) {
      id
      fields {
        name
        email
      }
      # Related contacts
      contacts {
        id
        fields {
          firstName
          lastName
          email
        }
      }
      # Related opportunities
      opportunities {
        id
        fields {
          title
          value
          stage
        }
      }
    }
  }
}
```

---

## Error Handling

GraphQL operations can return different types of errors:

### 1. GraphQL Validation Errors
```json
{
  "errors": [
    {
      "message": "Cannot query field \"invalidField\" on type \"Customer\"",
      "locations": [{"line": 5, "column": 9}],
      "extensions": {
        "code": "GRAPHQL_VALIDATION_FAILED"
      }
    }
  ]
}
```

### 2. Field Validation Errors
```json
{
  "data": {
    "Customer": {
      "create": {
        "validationErrors": [
          {
            "fieldName": "email",
            "errorType": "INVALID_EMAIL",
            "description": "Please enter a valid email address",
            "systemLabel": "Email"
          }
        ]
      }
    }
  }
}
```

### 3. Authentication Errors
```json
{
  "errors": [
    {
      "message": "Authentication failed",
      "extensions": {
        "code": "UNAUTHENTICATED"
      }
    }
  ]
}
```

---

## Performance Tips

1. **Request only needed fields**: Use GraphQL's field selection to minimize data transfer
2. **Use pagination**: Don't fetch all records at once
3. **Leverage caching**: GraphQL queries are cached based on the query structure
4. **Batch related data**: Use relational queries instead of multiple requests
5. **Use fragments**: Reuse common field selections with GraphQL fragments

---

<sub align="center">© 2025 InflowCRM &middot; GraphQL Operations Reference</sub>