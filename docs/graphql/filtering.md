# InflowCRM GraphQL API – Filtering Guide

This document provides comprehensive information about filtering capabilities in the InflowCRM GraphQL API, including all available operators and examples.

---

## Table of Contents

- [Filtering Overview](#filtering-overview)
- [Filter Structure](#filter-structure)
- [String Filtering](#string-filtering)
- [Number Filtering](#number-filtering)
- [Date Filtering](#date-filtering)
- [Boolean Filtering](#boolean-filtering)
- [Array Filtering](#array-filtering)
- [System Field Filtering](#system-field-filtering)
- [Complex Filtering](#complex-filtering)

---

## Filtering Overview

The InflowCRM GraphQL API provides powerful filtering capabilities through the `filtration` parameter. You can filter on:

- **Custom fields**: Any field with an API field name
- **System fields**: `id`, `createdTime`, `modifiedTime`
- **Multiple conditions**: Combine multiple filter conditions
- **Various operators**: Support for 12 different comparison operators

---

## Filter Structure

All filtering uses the `filtration` parameter with this structure:

```graphql
filtration: {
  fields: {
    # Custom field filters
    fieldName: { operator: value }
  }
  
  # System field filters (top-level)
  id: { operator: value }
  createdTime: { operator: value }
  modifiedTime: { operator: value }
}
```

---

## String Filtering

### Available Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `equal` | Exact match | `{ equal: "John Doe" }` |
| `notEqual` | Not equal | `{ notEqual: "Admin" }` |
| `in` | Value is in array | `{ in: ["Active", "Pending"] }` |
| `notIn` | Value is not in array | `{ notIn: ["Inactive", "Cancelled"] }` |

### Examples

**Exact Match:**
```graphql
query {
  Customer {
    getMany(
      perPage: 10
      page: 1
      filtration: {
        fields: {
          name: { equal: "John Doe" }
        }
      }
    ) {
      id
      fields {
        name
      }
    }
  }
}
```

**Multiple Values:**
```graphql
query {
  Customer {
    getMany(
      perPage: 15
      page: 1
      filtration: {
        fields: {
          companyName: { in: ["ACME Corp", "Tech Solutions", "Global Inc"] }
          status: { notIn: ["Cancelled", "Suspended"] }
        }
      }
    ) {
      id
      fields {
        name
        companyName
        status
      }
    }
  }
}
```

---

## Number Filtering

### Available Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `equal` | Exact match | `{ equal: 100 }` |
| `notEqual` | Not equal | `{ notEqual: 0 }` |
| `greater` | Greater than | `{ greater: 1000 }` |
| `greaterOrEqual` | Greater than or equal | `{ greaterOrEqual: 500 }` |
| `lower` | Less than | `{ lower: 10000 }` |
| `lowerOrEqual` | Less than or equal | `{ lowerOrEqual: 999 }` |
| `in` | Value is in array | `{ in: [100, 200, 300] }` |
| `notIn` | Value is not in array | `{ notIn: [0, -1] }` |

### Examples

**Range Filtering:**
```graphql
query {
  Customer {
    getMany(
      perPage: 25
      page: 1
      filtration: {
        fields: {
          annualRevenue: { greaterOrEqual: 1000000 }
          employeeCount: { lower: 500 }
          score: { greater: 7.5 }
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

**Specific Values:**
```graphql
query {
  Customer {
    getMany(
      perPage: 10
      page: 1
      filtration: {
        fields: {
          priority: { in: [1, 2, 3] }
          rating: { notEqual: 0 }
        }
      }
    ) {
      id
      fields {
        name
        priority
        rating
      }
    }
  }
}
```

---

## Date Filtering

### Available Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `equal` | Exact date match | `{ equal: "2023-01-15T00:00:00Z" }` |
| `notEqual` | Not equal | `{ notEqual: "2023-01-01T00:00:00Z" }` |
| `greater` | After date | `{ greater: "2023-01-01T00:00:00Z" }` |
| `greaterOrEqual` | On or after date | `{ greaterOrEqual: "2023-01-01T00:00:00Z" }` |
| `lower` | Before date | `{ lower: "2023-12-31T23:59:59Z" }` |
| `lowerOrEqual` | On or before date | `{ lowerOrEqual: "2023-12-31T23:59:59Z" }` |

### Examples

**Date Range:**
```graphql
query {
  Customer {
    getMany(
      perPage: 20
      page: 1
      filtration: {
        createdTime: { greater: "2023-01-01T00:00:00Z" }
        modifiedTime: { lower: "2023-12-31T23:59:59Z" }
      }
    ) {
      id
      createdTime
      modifiedTime
      fields {
        name
      }
    }
  }
}
```

**Recent Activity:**
```graphql
query {
  Customer {
    getMany(
      perPage: 30
      page: 1
      filtration: {
        fields: {
          lastContactDate: { greater: "2023-01-01T00:00:00Z" }
        }
      }
    ) {
      id
      fields {
        name
        lastContactDate
        nextFollowUpDate
      }
    }
  }
}
```

---

## Boolean Filtering

### Available Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `equal` | Exact boolean match | `{ equal: true }` |
| `notEqual` | Not equal | `{ notEqual: false }` |

### Examples

**Active Records:**
```graphql
query {
  Customer {
    getMany(
      perPage: 50
      page: 1
      filtration: {
        fields: {
          isActive: { equal: true }
          isVerified: { equal: true }
          isDeleted: { equal: false }
        }
      }
    ) {
      id
      fields {
        name
        isActive
        isVerified
        isDeleted
      }
    }
  }
}
```

---

## Array Filtering

### Multi-Select Fields

For fields that contain arrays (multi-select fields):

```graphql
query {
  Customer {
    getMany(
      perPage: 20
      page: 1
      filtration: {
        fields: {
          # Tags include "Enterprise"
          tags: { in: ["Enterprise"] }
          # Industry is one of these values
          industryCategories: { in: ["Technology", "Healthcare", "Finance"] }
        }
      }
    ) {
      id
      fields {
        name
        tags
        industryCategories
      }
    }
  }
}
```

---

## System Field Filtering

### Available System Fields

- `id` - Record ID
- `createdTime` - Creation timestamp
- `modifiedTime` - Last modification timestamp

### Examples

**Filter by ID:**
```graphql
query {
  Customer {
    getMany(
      perPage: 10
      page: 1
      filtration: {
        id: { in: ["507f1f77bcf86cd799439011", "507f1f77bcf86cd799439012"] }
      }
    ) {
      id
      fields {
        name
      }
    }
  }
}
```

**Recently Created:**
```graphql
query {
  Customer {
    getMany(
      perPage: 15
      page: 1
      filtration: {
        createdTime: { greater: "2023-01-01T00:00:00Z" }
      }
    ) {
      id
      createdTime
      fields {
        name
      }
    }
  }
}
```

**Recently Modified:**
```graphql
query {
  Customer {
    getMany(
      perPage: 20
      page: 1
      filtration: {
        modifiedTime: { greater: "2023-01-15T00:00:00Z" }
      }
    ) {
      id
      modifiedTime
      fields {
        name
      }
    }
  }
}
```

---

## Complex Filtering

### Multiple Conditions

Combine multiple filter conditions (AND logic):

```graphql
query {
  Customer {
    getMany(
      perPage: 30
      page: 1
      filtration: {
        fields: {
          # High-value enterprise customers
          annualRevenue: { greaterOrEqual: 5000000 }
          employeeCount: { greaterOrEqual: 1000 }
          industry: { in: ["Technology", "Healthcare", "Finance"] }
          status: { equal: "ACTIVE" }
          # Recently contacted
          lastContactDate: { greater: "2023-01-01T00:00:00Z" }
          # Has assigned sales rep
        }
        createdTime: { greater: "2023-01-01T00:00:00Z" }
      }
    ) {
      id
      createdTime
      fields {
        name
        annualRevenue
        employeeCount
        industry
        status
        lastContactDate
        salesRep
      }
    }
  }
}
```

### Qualification Criteria

```graphql
query GetQualifiedLeads {
  Customer {
    getMany(
      perPage: 25
      page: 1
      filtration: {
        fields: {
          # Lead qualification criteria
          annualRevenue: { greaterOrEqual: 1000000 }
          employeeCount: { greaterOrEqual: 50 }
          decisionMaker: { equal: true }
          budget: { greaterOrEqual: 100000 }
          timeframe: { in: ["0-3 months", "3-6 months"] }
          # Engagement level
          score: { greaterOrEqual: 7.0 }
          lastContactDate: { greater: "2023-01-01T00:00:00Z" }
        }
      }
    ) {
      id
      fields {
        name
        annualRevenue
        employeeCount
        decisionMaker
        budget
        timeframe
        score
        lastContactDate
      }
    }
  }
}
```

---

## Performance Considerations

### Indexing

- System fields (`id`, `createdTime`, `modifiedTime`) are automatically indexed
- Custom fields may have varying index performance
- Use specific filters rather than broad `contains` searches when possible

### Query Optimization

```graphql
# Good: Specific filters
query OptimizedQuery {
  Customer {
    getMany(
      perPage: 20
      page: 1
      filtration: {
        fields: {
          status: { equal: "ACTIVE" }
          industry: { in: ["Technology", "Healthcare"] }
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

# Less optimal: Broad text searches
```

---

## Common Patterns

### Status Filtering

```graphql
# Active records only
filtration: {
  fields: {
    status: { equal: "ACTIVE" }
    isDeleted: { equal: false }
  }
}

# Exclude certain statuses
filtration: {
  fields: {
    status: { notIn: ["CANCELLED", "SUSPENDED", "DELETED"] }
  }
}
```

---

<sub align="center">© 2025 InflowCRM &middot; GraphQL Filtering Guide</sub>