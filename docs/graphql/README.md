# InflowCRM GraphQL API Documentation

Welcome to the InflowCRM GraphQL API. This documentation provides everything you need to query and manipulate your CRM data using GraphQL's powerful, flexible, and type-safe approach.

---

## Quick Start

1. **Get your API Key** from your InflowCRM account admin.
2. **Set the GraphQL endpoint**:
   `https://srv.inflowcrm.pl/graphql`
3. **Authenticate every request** by adding your API key to the `x-api-key` header.
4. **Important:** Only fields with an **'Api field name'** set in the InflowCRM UI can be used in GraphQL operations.

Example (using `curl`):

```bash
curl -X POST https://srv.inflowcrm.pl/graphql \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{"query":"{ Customer { getMany(perPage: 10, page: 1) { id createdTime fields { name email } } } }"}'
```

---

## Authentication

All GraphQL operations require an API key via the `x-api-key` header.  
If the key is missing or invalid, a GraphQL authentication error is returned.

---

## GraphQL Endpoint

```
https://srv.inflowcrm.pl/graphql
```

---

## Rate Limits

- **300 requests per 5-minute window** per (API key + IP)
- Exceeding this limit returns HTTP 429 with a rate limit error

---

## Key Features

### Dynamic Schema Generation
- **Customer-specific schema**: Each customer gets a unique GraphQL schema based on their custom fields and modules
- **Real-time schema updates**: Schema reflects current field configurations
- **Type safety**: Full GraphQL type system with validation

### Core Operations
- **Queries**: `getOne`, `getOneById`, `getMany`, `getManyByIds`
- **Mutations**: `create`, `update`, `delete`
- **Flexible filtering**: Advanced filtering with multiple comparison operators
- **Pagination**: Built-in pagination support with `page` and `perPage`
- **Relational data**: Fetch related records in a single query

### Advanced Features
- **Union types**: Handle success responses and validation errors
- **Field projection**: Request only the fields you need
- **Validation feedback**: Detailed field-level validation errors
- **Relational queries**: Navigate between related modules seamlessly

---

## Schema Structure

### Per-Module Operations

Each module in your CRM (Customer, Contact, Note, etc.) gets its own set of GraphQL operations:

```graphql
type Query {
  Customer: CustomerQueries
  Contact: ContactQueries
  Note: NoteQueries
  # ... other modules
}

type Mutation {
  Customer: CustomerMutations
  Contact: ContactMutations
  Note: NoteMutations
  # ... other modules
}
```

### Common Fields

Every record includes these system fields:

```graphql
type Customer {
  id: ID!
  createdTime: Date!
  modifiedTime: Date!
  fields: CustomerFields!
  # relational fields...
}
```

---

## Common Response Patterns

### Successful Query Response
```json
{
  "data": {
    "Customer": {
      "getMany": [
        {
          "id": "507f1f77bcf86cd799439011",
          "createdTime": "2023-01-15T10:30:00Z",
          "modifiedTime": "2023-01-15T10:30:00Z",
          "fields": {
            "name": "John Doe",
            "email": "john@example.com"
          }
        }
      ]
    }
  }
}
```

### Validation Error Response
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

### GraphQL Error Response
```json
{
  "errors": [
    {
      "message": "Cannot query field \"invalidField\" on type \"Customer\"",
      "locations": [{"line": 2, "column": 3}],
      "extensions": {
        "code": "GRAPHQL_VALIDATION_FAILED"
      }
    }
  ]
}
```

---

## Available Modules

The available modules depend on your InflowCRM bundle configuration. Common modules include:

- **Customer** - Customer management
- **Contact** - Contact information  
- **Note** - Notes and comments
- **Opportunity** - Sales opportunities
- **Project** - Project management
- **Task** - Task tracking
- **User** - User management

> **Note:** Use the `getBundleModuleNames` REST API endpoint to get the exact list of available modules for your bundle.

---

## Getting Started

### 1. Introspect Your Schema

Use GraphQL introspection to explore your available operations:

```graphql
query {
  __schema {
    types {
      name
      kind
      fields {
        name
        type {
          name
        }
      }
    }
  }
}
```

### 2. List Available Operations

Query the root types to see what operations are available:

```graphql
query {
  __type(name: "Query") {
    fields {
      name
      description
      type {
        name
      }
    }
  }
}
```

### 3. Explore Module Structure

Get details about a specific module:

```graphql
query {
  __type(name: "Customer") {
    fields {
      name
      description
      type {
        name
      }
    }
  }
}
```

---

## Further Reading

- [Operations Reference](./operations.md) - Complete list of all available operations
- [Schema Guide](./schema.md) - Understanding the GraphQL schema structure
- [Query Examples](./queries.md) - Common query patterns and examples
- [Mutation Examples](./mutations.md) - Creating, updating, and deleting records
- [Filtering Guide](./filtering.md) - Advanced filtering capabilities
- [Error Handling](./errors.md) - Understanding and handling errors
- [Best Practices](./best-practices.md) - Performance tips and recommendations

---

## GraphQL vs REST

| Feature | GraphQL | REST |
|---------|---------|------|
| **Data Fetching** | Single request for multiple resources | Multiple requests often needed |
| **Over/Under Fetching** | Get exactly what you need | Fixed response structure |
| **Type Safety** | Strong typing with introspection | Documentation-based |
| **Caching** | Query-level caching | Resource-level caching |
| **Learning Curve** | Requires GraphQL knowledge | Familiar HTTP patterns |

Choose GraphQL when you need:
- Complex data relationships
- Mobile applications (bandwidth efficiency)
- Rapid frontend development
- Strong typing and validation

Choose REST when you need:
- Simple CRUD operations
- File uploads
- HTTP caching strategies
- Simpler integration patterns

---

<sub align="center">© 2025 InflowCRM &middot; GraphQL API Documentation</sub>