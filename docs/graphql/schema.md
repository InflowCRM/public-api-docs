# InflowCRM GraphQL API – Schema Guide

This document explains the GraphQL schema structure, type system, and how the dynamic schema is generated for each customer.

---

## Table of Contents

- [Schema Overview](#schema-overview)
- [Dynamic Schema Generation](#dynamic-schema-generation)
- [Core Types](#core-types)
- [Module Types](#module-types)
- [Input Types](#input-types)
- [Union Types](#union-types)
- [Scalar Types](#scalar-types)
- [Schema Introspection](#schema-introspection)

---

## Schema Overview

The InflowCRM GraphQL API uses a dynamically generated schema that is customized for each customer based on their:

- **Available modules** (Customer, Contact, Note, etc.)
- **Custom fields** configured in their CRM
- **Field permissions** and API field names
- **Relational mappings** between modules

This ensures that each customer sees only the data and operations relevant to their specific CRM configuration.

---

## Dynamic Schema Generation

### Schema Generation Process

1. **User Authentication**: API key is validated and user context is established
2. **Module Discovery**: Available modules are identified based on the user's bundle
3. **Field Retrieval**: Custom fields are loaded for each module
4. **Schema Building**: GraphQL types and operations are dynamically generated
5. **Schema Caching**: The generated schema is cached for performance

### Per-Customer Schemas

Each customer gets a unique schema because:
- **Custom fields differ** between customers
- **Module availability** depends on the bundle
- **Field names and types** are customer-specific
- **Relational mappings** vary by configuration

---

## Core Types

### Root Types

```graphql
type Query {
  # Dynamically generated per customer
  Customer: CustomerQueries
  Contact: ContactQueries
  Note: NoteQueries
  # ... other modules based on bundle
}

type Mutation {
  # Dynamically generated per customer
  Customer: CustomerMutations
  Contact: ContactMutations
  Note: NoteMutations
  # ... other modules based on bundle
}
```

### System Fields

All records include these standard fields:

```graphql
interface Record {
  id: ID!
  createdTime: Date!
  modifiedTime: Date!
}
```

---

## Module Types

### Basic Module Structure

Each module follows this pattern:

```graphql
type Customer implements Record {
  # System fields
  id: ID!
  createdTime: Date!
  modifiedTime: Date!
  
  # Custom fields (dynamically generated)
  fields: CustomerFields!
  
  # Relational fields (dynamically generated)
  contacts: [Contact!]!
  opportunities: [Opportunity!]!
}
```

### Custom Fields Type

Custom fields are grouped under a `fields` object:

```graphql
type CustomerFields {
  # Generated based on configured fields
  name: String
  email: String
  phone: String
  companySize: Int
  status: CustomerStatus  # Picklist field
  # ... other custom fields
}
```

### Query Operations Type

Each module gets its own query operations:

```graphql
type CustomerQueries {
  getOne(filtration: CustomerFiltration): Customer
  getOneById(id: ID!): Customer
  getMany(
    perPage: Int! = 20
    page: Int! = 1
    filtration: CustomerFiltration
  ): [Customer!]!
  getManyByIds(
    perPage: Int! = 20
    page: Int! = 1
    ids: [ID!]!
  ): [Customer!]!
}
```

### Mutation Operations Type

Each module gets its own mutation operations:

```graphql
type CustomerMutations {
  create(data: CustomerInputArgs!): CustomerAndValidationUnion!
  update(id: ID!, data: CustomerInputArgs!): CustomerAndValidationUnion!
  delete(id: ID!): OperationResult!
}
```

---

## Input Types

### Field Input Types

For mutations, custom fields are defined as input types:

```graphql
input CustomerFieldsInput {
  name: String
  email: String
  phone: String
  companySize: Int
  status: CustomerStatus
  # ... other custom fields
}
```

### Mutation Input Types

```graphql
input CustomerInputArgs {
  fields: CustomerFieldsInput
  # Relational fields for creating/linking related records
  contacts: [ContactRelationInput!]
  opportunities: [OpportunityRelationInput!]
}
```

### Filtration Input Types

```graphql
input CustomerFiltration {
  fields: CustomerFieldsFiltration
  systemFields: SystemFieldsFiltration
}

input CustomerFieldsFiltration {
  name: StringFilterCondition
  email: StringFilterCondition
  phone: StringFilterCondition
  companySize: IntFilterCondition
  status: CustomerStatusFilterCondition
  # ... other custom fields
}

input SystemFieldsFiltration {
  id: IDFilterCondition
  createdTime: DateFilterCondition
  modifiedTime: DateFilterCondition
}
```

---

## Union Types

### Success/Validation Union

Mutations return union types to handle both success and validation errors:

```graphql
union CustomerAndValidationUnion = Customer | ValidationError

type ValidationError {
  validationErrors: [FieldValidationError!]!
}

type FieldValidationError {
  fieldName: String!
  errorType: ValidationErrorType!
  description: String!
  systemLabel: String!
}
```

### Operation Result

Delete operations return a simple result:

```graphql
type OperationResult {
  success: Boolean!
  message: String!
}
```

---

## Scalar Types

### Built-in Scalars

```graphql
scalar ID      # Unique identifier
scalar String  # Text data
scalar Int     # Integer numbers
scalar Float   # Decimal numbers
scalar Boolean # True/false values
```

### Custom Scalars

```graphql
scalar Date    # ISO 8601 date strings
```

### Field Type Mapping

InflowCRM field types map to GraphQL types:

| CRM Field Type | GraphQL Type |
|----------------|--------------|
| Text | String |
| Number | Int or Float |
| Date | Date |
| Boolean | Boolean |
| Picklist | Custom Enum |
| Lookup | Related Type |
| Multi-select | [String!] |

---

## Schema Introspection

### Basic Introspection

Discover available types:

```graphql
query {
  __schema {
    types {
      name
      kind
      description
    }
  }
}
```

### Query Root Types

See available query operations:

```graphql
query {
  __type(name: "Query") {
    fields {
      name
      description
      type {
        name
        kind
      }
    }
  }
}
```

### Mutation Root Types

See available mutation operations:

```graphql
query {
  __type(name: "Mutation") {
    fields {
      name
      description
      type {
        name
        kind
      }
    }
  }
}
```

### Explore Module Types

Get details about a specific module:

```graphql
query {
  __type(name: "Customer") {
    name
    kind
    description
    fields {
      name
      description
      type {
        name
        kind
      }
    }
  }
}
```

### Explore Input Types

Understand mutation input structure:

```graphql
query {
  __type(name: "CustomerInputArgs") {
    name
    kind
    inputFields {
      name
      description
      type {
        name
        kind
      }
    }
  }
}
```

---

## Field Types and Validation

### String Fields

```graphql
type CustomerFields {
  name: String          # Optional text field
  email: String!        # Required text field
  description: String   # Long text field
}
```

### Number Fields

```graphql
type CustomerFields {
  age: Int             # Integer field
  revenue: Float       # Decimal field
  score: Int!          # Required integer
}
```

### Date Fields

```graphql
type CustomerFields {
  birthDate: Date      # Date field
  lastContact: Date    # DateTime field
}
```

### Boolean Fields

```graphql
type CustomerFields {
  isActive: Boolean    # True/false field
  isVip: Boolean!      # Required boolean
}
```

### Picklist Fields

```graphql
enum CustomerStatus {
  ACTIVE
  INACTIVE
  PENDING
  CANCELLED
}

type CustomerFields {
  status: CustomerStatus    # Single select
  tags: [String!]          # Multi-select
}
```

### Lookup/Relation Fields

```graphql
type Customer {
  # ... other fields
  primaryContact: Contact      # Single lookup
  contacts: [Contact!]!        # Multiple lookup
  assignedUser: User           # User lookup
}
```

---

## Schema Caching

### Cache Keys

Schemas are cached using keys that include:
- Customer/tenant ID
- Module configurations hash
- Field configurations hash
- Last modification timestamp

### Cache Invalidation

The schema cache is invalidated when:
- Custom fields are added/modified/removed
- Module configurations change
- Field permissions are updated
- API field names are changed

### Performance Benefits

- **Faster response times**: Cached schemas avoid regeneration
- **Reduced CPU usage**: Schema building is expensive
- **Better scalability**: Multiple requests use the same cached schema

---

## Best Practices

### Schema Design

1. **Use meaningful field names**: Choose clear, descriptive API field names
2. **Leverage GraphQL types**: Use appropriate scalar types for validation
3. **Document your fields**: Add descriptions for better developer experience
4. **Plan relationships**: Design relational mappings carefully

### Field Configuration

1. **Set API field names**: Only fields with API field names appear in GraphQL
2. **Use consistent naming**: Follow camelCase conventions
3. **Consider field types**: Choose appropriate field types for GraphQL mapping
4. **Document field purpose**: Add descriptions for complex fields

### Performance Optimization

1. **Use field selection**: Request only needed fields
2. **Implement pagination**: Use `perPage` and `page` parameters
3. **Cache frequently used queries**: Leverage GraphQL query caching
4. **Monitor schema size**: Large schemas can impact performance

---

<sub align="center">© 2025 InflowCRM &middot; GraphQL Schema Guide</sub>