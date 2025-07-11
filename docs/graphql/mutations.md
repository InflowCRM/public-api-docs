# InflowCRM GraphQL API – Mutation Examples

This document provides practical examples of GraphQL mutations for creating, updating, and deleting records in the InflowCRM API.

---

## Table of Contents

- [Create Operations](#create-operations)
- [Update Operations](#update-operations)
- [Delete Operations](#delete-operations)
- [Batch Operations](#batch-operations)
- [Relational Mutations](#relational-mutations)
- [Error Handling](#error-handling)

---

## Create Operations

### Basic Record Creation

```graphql
mutation CreateCustomer {
  Customer {
    create(data: {
      fields: {
        name: "John Doe"
        email: "john@example.com"
        phone: "+1234567890"
        companyName: "ACME Corporation"
      }
    }) {
      ... on Customer {
        id
        createdTime
        fields {
          name
          email
          phone
          companyName
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
          "name": "John Doe",
          "email": "john@example.com",
          "phone": "+1234567890",
          "companyName": "ACME Corporation"
        }
      }
    }
  }
}
```

### Create with All Field Types

```graphql
mutation CreateCustomerWithAllFields {
  Customer {
    create(data: {
      fields: {
        # Text fields
        name: "Alice Johnson"
        email: "alice@example.com"
        phone: "+1987654321"
        
        # Number fields
        annualRevenue: 2500000
        employeeCount: 150
        score: 8.5
        
        # Date fields
        lastContactDate: "2023-01-15T00:00:00Z"
        
        # Boolean fields
        isActive: true
        isVip: false
        
        # Picklist fields
        status: "ACTIVE"
        industry: "Technology"
        
        # Multi-select fields
        tags: ["Enterprise", "Strategic", "High-Value"]
      }
    }) {
      ... on Customer {
        id
        createdTime
        fields {
          name
          email
          phone
          annualRevenue
          employeeCount
          score
          lastContactDate
          isActive
          isVip
          status
          industry
          tags
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

### Create Contact

```graphql
mutation CreateContact {
  Contact {
    create(data: {
      fields: {
        firstName: "Sarah"
        lastName: "Williams"
        email: "sarah.williams@example.com"
        phone: "+1555123456"
        position: "VP of Sales"
        department: "Sales"
      }
    }) {
      ... on Contact {
        id
        createdTime
        fields {
          firstName
          lastName
          email
          phone
          position
          department
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

### Create Opportunity

```graphql
mutation CreateOpportunity {
  Opportunity {
    create(data: {
      fields: {
        title: "Enterprise Software License"
        value: 150000
        stage: "PROPOSAL"
        closeDate: "2023-06-30T00:00:00Z"
        probability: 75
        description: "Large enterprise software licensing opportunity"
      }
    }) {
      ... on Opportunity {
        id
        createdTime
        fields {
          title
          value
          stage
          closeDate
          probability
          description
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

---

## Update Operations

### Basic Record Update

```graphql
mutation UpdateCustomer {
  Customer {
    update(
      id: "507f1f77bcf86cd799439011"
      data: {
        fields: {
          name: "John Doe Updated"
          email: "john.doe.updated@example.com"
          status: "ACTIVE"
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

### Partial Field Update

```graphql
mutation UpdateCustomerStatus {
  Customer {
    update(
      id: "507f1f77bcf86cd799439011"
      data: {
        fields: {
          status: "INACTIVE"
          lastContactDate: "2023-01-17T10:30:00Z"
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
          lastContactDate
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

### Update Contact Information

```graphql
mutation UpdateContact {
  Contact {
    update(
      id: "507f1f77bcf86cd799439020"
      data: {
        fields: {
          phone: "+1555987654"
          position: "Senior VP of Sales"
          department: "Sales & Marketing"
        }
      }
    ) {
      ... on Contact {
        id
        modifiedTime
        fields {
          firstName
          lastName
          email
          phone
          position
          department
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

### Update Opportunity Stage

```graphql
mutation UpdateOpportunityStage {
  Opportunity {
    update(
      id: "507f1f77bcf86cd799439030"
      data: {
        fields: {
          stage: "NEGOTIATION"
          probability: 85
          value: 175000
        }
      }
    ) {
      ... on Opportunity {
        id
        modifiedTime
        fields {
          title
          value
          stage
          probability
          closeDate
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

---

## Delete Operations

### Delete Customer

```graphql
mutation DeleteCustomer {
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

### Delete Contact

```graphql
mutation DeleteContact {
  Contact {
    delete(id: "507f1f77bcf86cd799439020") {
      result
    }
  }
}
```

### Delete Opportunity

```graphql
mutation DeleteOpportunity {
  Opportunity {
    delete(id: "507f1f77bcf86cd799439030") {
      result
    }
  }
}
```

---

## Batch Operations

### Create Multiple Records

```graphql
mutation CreateMultipleCustomers {
  customer1: Customer {
    create(data: {
      fields: {
        name: "Customer One"
        email: "customer1@example.com"
        phone: "+1111111111"
      }
    }) {
      ... on Customer {
        id
        fields {
          name
          email
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
  
  customer2: Customer {
    create(data: {
      fields: {
        name: "Customer Two"
        email: "customer2@example.com"
        phone: "+2222222222"
      }
    }) {
      ... on Customer {
        id
        fields {
          name
          email
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

### Update Multiple Records

```graphql
mutation UpdateMultipleCustomers {
  customer1: Customer {
    update(
      id: "507f1f77bcf86cd799439011"
      data: {
        fields: {
          status: "ACTIVE"
        }
      }
    ) {
      ... on Customer {
        id
        fields {
          name
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
  
  customer2: Customer {
    update(
      id: "507f1f77bcf86cd799439012"
      data: {
        fields: {
          status: "INACTIVE"
        }
      }
    ) {
      ... on Customer {
        id
        fields {
          name
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

---

## Relational Mutations

### Create Customer with Related Contact

```graphql
mutation CreateCustomerWithContact {
  # First create the customer
  customer: Customer {
    create(data: {
      fields: {
        name: "Tech Solutions Inc"
        email: "info@techsolutions.com"
        phone: "+1555000000"
      }
    }) {
      ... on Customer {
        id
        fields {
          name
          email
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
  
  # Then create the contact (you'll need to link them separately)
  contact: Contact {
    create(data: {
      fields: {
        firstName: "Mike"
        lastName: "Johnson"
        email: "mike.johnson@techsolutions.com"
        phone: "+1555000001"
        position: "CTO"
      }
    }) {
      ... on Contact {
        id
        fields {
          firstName
          lastName
          email
          phone
          position
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

### Create Opportunity for Customer

```graphql
mutation CreateOpportunityForCustomer {
  Opportunity {
    create(data: {
      fields: {
        title: "Cloud Migration Project"
        value: 500000
        stage: "QUALIFICATION"
        closeDate: "2023-09-30T00:00:00Z"
        probability: 30
        description: "Large-scale cloud migration project"
        # Link to customer (exact field name depends on your schema)
        customerId: "507f1f77bcf86cd799439011"
      }
    }) {
      ... on Opportunity {
        id
        fields {
          title
          value
          stage
          closeDate
          probability
          customerId
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

---

## Error Handling

### Required Field Validation

```graphql
mutation CreateCustomerMissingRequiredField {
  Customer {
    create(data: {
      fields: {
        # Missing required 'name' field
        email: "incomplete@example.com"
      }
    }) {
      ... on Customer {
        id
        fields {
          name
          email
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

**Validation Error Response:**
```json
{
  "data": {
    "Customer": {
      "create": {
        "validationErrors": [
          {
            "fieldName": "name",
            "errorType": "REQUIRED",
            "description": "Name is required",
            "systemLabel": "Customer Name"
          }
        ]
      }
    }
  }
}
```

### Invalid Email Format

```graphql
mutation CreateCustomerInvalidEmail {
  Customer {
    create(data: {
      fields: {
        name: "Test Customer"
        email: "invalid-email-format"
      }
    }) {
      ... on Customer {
        id
        fields {
          name
          email
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

**Validation Error Response:**
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

### Multiple Validation Errors

```graphql
mutation CreateCustomerMultipleErrors {
  Customer {
    create(data: {
      fields: {
        # Missing required name
        email: "invalid-email"  # Invalid email format
        annualRevenue: -1000    # Invalid negative value
      }
    }) {
      ... on Customer {
        id
        fields {
          name
          email
          annualRevenue
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

**Multiple Validation Errors Response:**
```json
{
  "data": {
    "Customer": {
      "create": {
        "validationErrors": [
          {
            "fieldName": "name",
            "errorType": "REQUIRED",
            "description": "Name is required",
            "systemLabel": "Customer Name"
          },
          {
            "fieldName": "email",
            "errorType": "INVALID_EMAIL",
            "description": "Please enter a valid email address",
            "systemLabel": "Email"
          },
          {
            "fieldName": "annualRevenue",
            "errorType": "INVALID_VALUE",
            "description": "Annual revenue must be a positive number",
            "systemLabel": "Annual Revenue"
          }
        ]
      }
    }
  }
}
```

### Update Non-Existent Record

```graphql
mutation UpdateNonExistentCustomer {
  Customer {
    update(
      id: "nonexistent-id"
      data: {
        fields: {
          name: "Updated Name"
        }
      }
    ) {
      ... on Customer {
        id
        fields {
          name
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

**Error Response:**
```json
{
  "errors": [
    {
      "message": "Record not found",
      "extensions": {
        "code": "RECORD_NOT_FOUND"
      }
    }
  ]
}
```

---

## Using Variables in Mutations

### Create with Variables

```graphql
mutation CreateCustomerWithVariables($customerData: CustomerFieldsInput!) {
  Customer {
    create(data: { fields: $customerData }) {
      ... on Customer {
        id
        createdTime
        fields {
          name
          email
          phone
          companyName
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

**Variables:**
```json
{
  "customerData": {
    "name": "Variable Customer",
    "email": "variable@example.com",
    "phone": "+1555000999",
    "companyName": "Variables Inc"
  }
}
```

### Update with Variables

```graphql
mutation UpdateCustomerWithVariables($id: ID!, $updateData: CustomerFieldsInput!) {
  Customer {
    update(id: $id, data: { fields: $updateData }) {
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

**Variables:**
```json
{
  "id": "507f1f77bcf86cd799439011",
  "updateData": {
    "name": "Updated Customer Name",
    "status": "ACTIVE"
  }
}
```

---

## Best Practices

### 1. Always Handle Union Types

```graphql
mutation CreateCustomerSafely {
  Customer {
    create(data: {
      fields: {
        name: "Safe Customer"
        email: "safe@example.com"
      }
    }) {
      # Always handle both success and error cases
      ... on Customer {
        id
        createdTime
        fields {
          name
          email
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

### 2. Use Descriptive Operation Names

```graphql
mutation CreateNewCustomerFromWebForm {
  Customer {
    create(data: {
      fields: {
        name: "Web Form Customer"
        email: "webform@example.com"
        source: "Website"
      }
    }) {
      ... on Customer {
        id
        fields {
          name
          email
          source
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

### 3. Request Only Needed Fields

```graphql
mutation UpdateCustomerMinimal {
  Customer {
    update(
      id: "507f1f77bcf86cd799439011"
      data: {
        fields: {
          status: "ACTIVE"
        }
      }
    ) {
      ... on Customer {
        id
        fields {
          status  # Only request the updated field
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

### 4. Use Fragments for Consistency

```graphql
fragment CustomerValidationErrors on ValidationError {
  validationErrors {
    fieldName
    errorType
    description
    systemLabel
  }
}

mutation CreateCustomerWithFragment {
  Customer {
    create(data: {
      fields: {
        name: "Fragment Customer"
        email: "fragment@example.com"
      }
    }) {
      ... on Customer {
        id
        createdTime
        fields {
          name
          email
        }
      }
      ...CustomerValidationErrors
    }
  }
}
```

---

<sub align="center">© 2025 InflowCRM &middot; GraphQL Mutation Examples</sub>