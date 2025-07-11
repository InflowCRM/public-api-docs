# InflowCRM GraphQL API – Query Examples

This document provides practical examples of common GraphQL query patterns and use cases for the InflowCRM API.

---

## Table of Contents

- [Basic Queries](#basic-queries)
- [Filtering Examples](#filtering-examples)
- [Pagination Examples](#pagination-examples)
- [Relational Queries](#relational-queries)
- [Complex Queries](#complex-queries)
- [Performance Optimization](#performance-optimization)

---

## Basic Queries

### Get Single Record by ID

```graphql
query GetCustomerById {
  Customer {
    getOneById(id: "507f1f77bcf86cd799439011") {
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

### Get Single Record by Filter

```graphql
query GetCustomerByEmail {
  Customer {
    getOne(filtration: { 
      fields: { 
        email: { equal: "john@example.com" } 
      } 
    }) {
      id
      fields {
        name
        email
        companyName
      }
    }
  }
}
```

### Get Multiple Records

```graphql
query GetCustomers {
  Customer {
    getMany(perPage: 10, page: 1) {
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

### Get Multiple Records by IDs

```graphql
query GetCustomersByIds {
  Customer {
    getManyByIds(
      perPage: 5
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

---

## Filtering Examples

### String Filtering

```graphql
query FilterCustomersByName {
  Customer {
    getMany(
      perPage: 20
      page: 1
      filtration: {
        fields: {
          name: { contains: "John" }
          email: { notEqual: "admin@example.com" }
          companyName: { in: ["ACME Corp", "Tech Solutions"] }
        }
      }
    ) {
      id
      fields {
        name
        email
        companyName
      }
    }
  }
}
```

### Number Filtering

```graphql
query FilterCustomersByRevenue {
  Customer {
    getMany(
      perPage: 15
      page: 1
      filtration: {
        fields: {
          annualRevenue: { greaterOrEqual: 100000 }
          employeeCount: { lower: 500 }
          score: { in: [8, 9, 10] }
        }
      }
    ) {
      id
      fields {
        name
        annualRevenue
        employeeCount
        score
      }
    }
  }
}
```

### Date Filtering

```graphql
query FilterCustomersByDate {
  Customer {
    getMany(
      perPage: 10
      page: 1
      filtration: {
        systemFields: {
          createdTime: { greater: "2023-01-01T00:00:00Z" }
          modifiedTime: { lower: "2023-12-31T23:59:59Z" }
        }
        fields: {
          lastContactDate: { isNotNull: true }
        }
      }
    ) {
      id
      createdTime
      modifiedTime
      fields {
        name
        lastContactDate
      }
    }
  }
}
```

### Boolean and Null Filtering

```graphql
query FilterActiveCustomers {
  Customer {
    getMany(
      perPage: 25
      page: 1
      filtration: {
        fields: {
          isActive: { equal: true }
          isVip: { equal: false }
          description: { isNull: false }
        }
      }
    ) {
      id
      fields {
        name
        isActive
        isVip
        description
      }
    }
  }
}
```

### Picklist Filtering

```graphql
query FilterCustomersByStatus {
  Customer {
    getMany(
      perPage: 30
      page: 1
      filtration: {
        fields: {
          status: { in: [ACTIVE, PENDING] }
          industry: { notEqual: "Technology" }
        }
      }
    ) {
      id
      fields {
        name
        status
        industry
      }
    }
  }
}
```

---

## Pagination Examples

### Basic Pagination

```graphql
query GetCustomersPage2 {
  Customer {
    getMany(perPage: 20, page: 2) {
      id
      fields {
        name
        email
      }
    }
  }
}
```

### Large Page Size

```graphql
query GetManyCustomers {
  Customer {
    getMany(perPage: 100, page: 1) {
      id
      fields {
        name
        email
        phone
      }
    }
  }
}
```

### Paginated with Filtering

```graphql
query GetActiveCustomersPage3 {
  Customer {
    getMany(
      perPage: 15
      page: 3
      filtration: {
        fields: {
          status: { equal: "ACTIVE" }
        }
      }
    ) {
      id
      fields {
        name
        email
        status
      }
    }
  }
}
```

---

## Relational Queries

> **Dynamic fields** – For every relationship, the API exposes a field named `getRelated<Module>` (e.g., `getRelatedContacts`, `getRelatedOpportunities`).
> The `<Module>` part is generated from the translated singular name of the target module, capitalised and concatenated. These fields accept the same arguments as the standard list queries (`perPage`, `page`, `filtration`, etc.) and return a paginated list of related records.

### Customer with Contacts

```graphql
query GetCustomerWithContacts {
  Customer {
    getOneById(id: "507f1f77bcf86cd799439011") {
      id
      fields {
        name
        email
      }
      contacts {
        id
        fields {
          firstName
          lastName
          email
          phone
        }
      }
    }
  }
}
```

### Customer with Opportunities

```graphql
query GetCustomerWithOpportunities {
  Customer {
    getOneById(id: "507f1f77bcf86cd799439011") {
      id
      fields {
        name
        companyName
      }
      opportunities {
        id
        fields {
          title
          value
          stage
          closeDate
        }
      }
    }
  }
}
```

### Multiple Relationships

```graphql
query GetCustomerWithAllRelations {
  Customer {
    getOneById(id: "507f1f77bcf86cd799439011") {
      id
      fields {
        name
        email
        companyName
      }
      contacts {
        id
        fields {
          firstName
          lastName
          email
        }
      }
      opportunities {
        id
        fields {
          title
          value
          stage
        }
      }
      notes {
        id
        fields {
          title
          content
        }
      }
    }
  }
}
```

### Nested Relationships

```graphql
query GetCustomerWithNestedData {
  Customer {
    getOneById(id: "507f1f77bcf86cd799439011") {
      id
      fields {
        name
      }
      contacts {
        id
        fields {
          firstName
          lastName
        }
        # Get notes for each contact
        notes {
          id
          fields {
            title
            content
          }
        }
      }
    }
  }
}
```

### Customer – Related Opportunities (dynamic field)

```graphql
query CustomerRelatedOpportunities {
  Customer {
    getRelatedOpportunities(id: "507f1f77bcf86cd799439011", perPage: 5, page: 1) {
      id
      fields {
        title
        value
        stage
      }
    }
  }
}
```

---

## Complex Queries

### Multiple Modules in One Query

```graphql
query GetDashboardData {
  Customer {
    getMany(perPage: 5, page: 1) {
      id
      fields {
        name
        email
        status
      }
    }
  }
  
  Opportunity {
    getMany(
      perPage: 10
      page: 1
      filtration: {
        fields: {
          stage: { in: ["PROPOSAL", "NEGOTIATION"] }
        }
      }
    ) {
      id
      fields {
        title
        value
        stage
      }
    }
  }
  
  Note {
    getMany(
      perPage: 3
      page: 1
      filtration: {
        systemFields: {
          createdTime: { greater: "2023-01-01T00:00:00Z" }
        }
      }
    ) {
      id
      fields {
        title
        content
      }
    }
  }
}
```

### Complex Filtering with Multiple Conditions

```graphql
query GetQualifiedLeads {
  Customer {
    getMany(
      perPage: 20
      page: 1
      filtration: {
        fields: {
          # High-value prospects
          annualRevenue: { greaterOrEqual: 1000000 }
          employeeCount: { greaterOrEqual: 50 }
          # Active and engaged
          status: { equal: "ACTIVE" }
          lastContactDate: { greater: "2023-01-01T00:00:00Z" }
          # Not already customers
          customerType: { notEqual: "EXISTING" }
        }
      }
    ) {
      id
      fields {
        name
        companyName
        annualRevenue
        employeeCount
        status
        lastContactDate
        customerType
      }
    }
  }
}
```

### Search Query

```graphql
query SearchCustomers($searchTerm: String!) {
  Customer {
    getMany(
      perPage: 50
      page: 1
      filtration: {
        fields: {
          name: { contains: $searchTerm }
          companyName: { contains: $searchTerm }
          email: { contains: $searchTerm }
        }
      }
    ) {
      id
      fields {
        name
        companyName
        email
        phone
      }
    }
  }
}
```

---

## Performance Optimization

### Minimal Field Selection

```graphql
query GetCustomerNames {
  Customer {
    getMany(perPage: 100, page: 1) {
      id
      fields {
        name  # Only get the name field
      }
    }
  }
}
```

### Selective Relational Data

```graphql
query GetCustomerWithLimitedContacts {
  Customer {
    getOneById(id: "507f1f77bcf86cd799439011") {
      id
      fields {
        name
        email
      }
      # Only get basic contact info
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

### Paginated Relationships

```graphql
query GetCustomerWithPaginatedOpportunities {
  Customer {
    getOneById(id: "507f1f77bcf86cd799439011") {
      id
      fields {
        name
      }
    }
  }
  
  # Separate query for opportunities with pagination
  Opportunity {
    getMany(
      perPage: 10
      page: 1
      filtration: {
        fields: {
          customerId: { equal: "507f1f77bcf86cd799439011" }
        }
      }
    ) {
      id
      fields {
        title
        value
        stage
      }
    }
  }
}
```

---

## Query Variables

### Using Variables for Filtering

```graphql
query GetCustomersByStatus($status: String!, $perPage: Int = 20) {
  Customer {
    getMany(
      perPage: $perPage
      page: 1
      filtration: {
        fields: {
          status: { equal: $status }
        }
      }
    ) {
      id
      fields {
        name
        email
        status
      }
    }
  }
}
```

**Variables:**
```json
{
  "status": "ACTIVE",
  "perPage": 25
}
```

### Dynamic ID Lookup

```graphql
query GetCustomerById($customerId: ID!) {
  Customer {
    getOneById(id: $customerId) {
      id
      fields {
        name
        email
        companyName
      }
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

**Variables:**
```json
{
  "customerId": "507f1f77bcf86cd799439011"
}
```

---

## GraphQL Fragments

### Reusable Customer Fields

```graphql
fragment CustomerBasicInfo on Customer {
  id
  createdTime
  modifiedTime
  fields {
    name
    email
    phone
    companyName
  }
}

query GetCustomers {
  Customer {
    getMany(perPage: 10, page: 1) {
      ...CustomerBasicInfo
    }
  }
}

query GetCustomerById {
  Customer {
    getOneById(id: "507f1f77bcf86cd799439011") {
      ...CustomerBasicInfo
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

### Contact Fragment

```graphql
fragment ContactInfo on Contact {
  id
  createdTime
  fields {
    firstName
    lastName
    email
    phone
    position
  }
}

query GetCustomerWithContacts {
  Customer {
    getOneById(id: "507f1f77bcf86cd799439011") {
      id
      fields {
        name
      }
      contacts {
        ...ContactInfo
      }
    }
  }
}
```

---

## Error Handling in Queries

### Handling Non-Existent Records

```graphql
query GetCustomerSafely {
  Customer {
    getOneById(id: "nonexistent-id") {
      id
      fields {
        name
        email
      }
    }
  }
}
```

**Response (null result):**
```json
{
  "data": {
    "Customer": {
      "getOneById": null
    }
  }
}
```

### Invalid Field Names

```graphql
query GetCustomerWithInvalidField {
  Customer {
    getOneById(id: "507f1f77bcf86cd799439011") {
      id
      fields {
        name
        invalidField  # This will cause an error
      }
    }
  }
}
```

**Error Response:**
```json
{
  "errors": [
    {
      "message": "Cannot query field \"invalidField\" on type \"CustomerFields\"",
      "locations": [{"line": 6, "column": 9}],
      "extensions": {
        "code": "GRAPHQL_VALIDATION_FAILED"
      }
    }
  ]
}
```

---

<sub align="center">© 2025 InflowCRM &middot; GraphQL Query Examples</sub>