# InflowCRM GraphQL API – Error Handling

This document explains the different types of errors that can occur when using the InflowCRM GraphQL API and how to handle them effectively.

---

## Table of Contents

- [Error Types Overview](#error-types-overview)
- [GraphQL Validation Errors](#graphql-validation-errors)
- [Field Validation Errors](#field-validation-errors)
- [Authentication Errors](#authentication-errors)
- [Authorization Errors](#authorization-errors)
- [System Errors](#system-errors)
- [Network Errors](#network-errors)
- [Error Handling Best Practices](#error-handling-best-practices)

---

## Error Types Overview

The InflowCRM GraphQL API can return several types of errors:

1. **GraphQL Validation Errors** - Query syntax or schema validation issues
2. **Field Validation Errors** - Business logic validation failures
3. **Authentication Errors** - Missing or invalid API keys
4. **Authorization Errors** - Insufficient permissions
5. **System Errors** - Server-side issues
6. **Network Errors** - Connection or timeout issues

---

## GraphQL Validation Errors

These errors occur when the GraphQL query itself is invalid according to the schema or GraphQL specification.

### Common Validation Errors

#### Invalid Field Names

```graphql
query {
  Customer {
    getOneById(id: "507f1f77bcf86cd799439011") {
      id
      fields {
        name
        invalidField  # This field doesn't exist
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
      "locations": [
        {
          "line": 7,
          "column": 9
        }
      ],
      "extensions": {
        "code": "GRAPHQL_VALIDATION_FAILED"
      }
    }
  ]
}
```

#### Missing Required Arguments

```graphql
query {
  Customer {
    getOneById {  # Missing required 'id' argument
      id
      fields {
        name
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
      "message": "Field \"getOneById\" argument \"id\" of type \"ID!\" is required, but it was not provided",
      "locations": [
        {
          "line": 3,
          "column": 5
        }
      ],
      "extensions": {
        "code": "GRAPHQL_VALIDATION_FAILED"
      }
    }
  ]
}
```

#### Invalid Argument Types

```graphql
query {
  Customer {
    getOneById(id: 123) {  # ID should be a string
      id
      fields {
        name
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
      "message": "Expected type \"ID\", found 123",
      "locations": [
        {
          "line": 3,
          "column": 20
        }
      ],
      "extensions": {
        "code": "GRAPHQL_VALIDATION_FAILED"
      }
    }
  ]
}
```

#### Invalid Query Structure

```graphql
query {
  Customer {
    getMany(perPage: 10) {
      id
      fields {
        name
      }
    }
  }
  # Missing closing brace
```

**Error Response:**
```json
{
  "errors": [
    {
      "message": "Syntax Error: Expected Name, found EOF",
      "locations": [
        {
          "line": 10,
          "column": 1
        }
      ],
      "extensions": {
        "code": "GRAPHQL_PARSE_FAILED"
      }
    }
  ]
}
```

### Handling GraphQL Validation Errors

```javascript
// JavaScript example
const response = await fetch('/graphql', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-api-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({ query: yourQuery })
});

const result = await response.json();

if (result.errors) {
  result.errors.forEach(error => {
    if (error.extensions?.code === 'GRAPHQL_VALIDATION_FAILED') {
      console.error('GraphQL validation error:', error.message);
      console.error('Location:', error.locations);
    }
  });
}
```

---

## Field Validation Errors

These errors occur when business logic validation fails during mutations. They are returned as part of the data using union types.

### Common Validation Error Types

| Error Type | Description | Example |
|------------|-------------|---------|
| `REQUIRED` | Field is required but not provided | Name field is empty |
| `INVALID_EMAIL` | Invalid email format | "not-an-email" |
| `INVALID_PHONE` | Invalid phone number format | "123" |
| `INVALID_DATE` | Invalid date format | "not-a-date" |
| `INVALID_VALUE` | Value doesn't meet constraints | Negative revenue |
| `DUPLICATE_VALUE` | Value already exists | Duplicate email |
| `FIELD_TOO_LONG` | Field exceeds maximum length | Description too long |
| `FIELD_TOO_SHORT` | Field below minimum length | Password too short |
| `INVALID_CHOICE` | Invalid picklist value | Status not in enum |
| `INVALID_RANGE` | Value outside valid range | Score > 10 |

### Examples

#### Required Field Validation

```graphql
mutation {
  Customer {
    create(data: {
      fields: {
        # Missing required 'name' field
        email: "john@example.com"
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
            "description": "Customer name is required",
            "systemLabel": "Customer Name"
          }
        ]
      }
    }
  }
}
```

#### Multiple Validation Errors

```graphql
mutation {
  Customer {
    create(data: {
      fields: {
        name: ""  # Empty required field
        email: "invalid-email"  # Invalid format
        annualRevenue: -1000  # Invalid value
        status: "INVALID_STATUS"  # Invalid enum value
      }
    }) {
      ... on Customer {
        id
        fields {
          name
          email
          annualRevenue
          status
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
            "description": "Customer name is required",
            "systemLabel": "Customer Name"
          },
          {
            "fieldName": "email",
            "errorType": "INVALID_EMAIL",
            "description": "Please enter a valid email address",
            "systemLabel": "Email Address"
          },
          {
            "fieldName": "annualRevenue",
            "errorType": "INVALID_VALUE",
            "description": "Annual revenue must be a positive number",
            "systemLabel": "Annual Revenue"
          },
          {
            "fieldName": "status",
            "errorType": "INVALID_CHOICE",
            "description": "Please select a valid status",
            "systemLabel": "Status"
          }
        ]
      }
    }
  }
}
```

### Handling Field Validation Errors

```javascript
// JavaScript example
function handleMutationResponse(response) {
  if (response.data?.Customer?.create) {
    const result = response.data.Customer.create;
    
    if (result.validationErrors) {
      // Handle validation errors
      result.validationErrors.forEach(error => {
        console.error(`${error.systemLabel}: ${error.description}`);
        
        // Display error next to specific field
        displayFieldError(error.fieldName, error.description);
      });
    } else {
      // Success - result is the created Customer
      console.log('Customer created:', result.id);
      displaySuccess('Customer created successfully');
    }
  }
}

function displayFieldError(fieldName, message) {
  const field = document.getElementById(fieldName);
  if (field) {
    field.classList.add('error');
    const errorElement = field.parentNode.querySelector('.error-message');
    if (errorElement) {
      errorElement.textContent = message;
    }
  }
}
```

---

## Authentication Errors

These errors occur when the API key is missing, invalid, or expired.

### Missing API Key

```bash
curl -X POST https://srv.inflowcrm.pl/graphql \
  -H "Content-Type: application/json" \
  -d '{"query":"{ Customer { getMany(perPage: 10) { id } } }"}'
```

**Error Response:**
```json
{
  "errors": [
    {
      "message": "Authentication required",
      "extensions": {
        "code": "UNAUTHENTICATED"
      }
    }
  ]
}
```

### Invalid API Key

```bash
curl -X POST https://srv.inflowcrm.pl/graphql \
  -H "Content-Type: application/json" \
  -H "x-api-key: invalid-key" \
  -d '{"query":"{ Customer { getMany(perPage: 10) { id } } }"}'
```

**Error Response:**
```json
{
  "errors": [
    {
      "message": "Invalid API key",
      "extensions": {
        "code": "UNAUTHENTICATED"
      }
    }
  ]
}
```

### Handling Authentication Errors

```javascript
function handleAuthenticationError(error) {
  if (error.extensions?.code === 'UNAUTHENTICATED') {
    // Redirect to login or show authentication error
    redirectToLogin();
    showError('Please log in to access this resource');
  }
}
```

---

## Authorization Errors

These errors occur when the user doesn't have permission to perform the requested operation.

### Insufficient Permissions

```graphql
mutation {
  Customer {
    delete(id: "507f1f77bcf86cd799439011") {
      success
      message
    }
  }
}
```

**Error Response:**
```json
{
  "errors": [
    {
      "message": "Insufficient permissions to delete customer records",
      "extensions": {
        "code": "FORBIDDEN"
      }
    }
  ]
}
```

### Module Access Denied

```graphql
query {
  AdminModule {
    getMany(perPage: 10) {
      id
    }
  }
}
```

**Error Response:**
```json
{
  "errors": [
    {
      "message": "Access denied to AdminModule",
      "extensions": {
        "code": "FORBIDDEN"
      }
    }
  ]
}
```

---

## System Errors

These errors occur due to server-side issues or unexpected conditions.

### Record Not Found

```graphql
query {
  Customer {
    getOneById(id: "nonexistent-id") {
      id
      fields {
        name
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

### Server Error

```json
{
  "errors": [
    {
      "message": "Internal server error",
      "extensions": {
        "code": "INTERNAL_SERVER_ERROR"
      }
    }
  ]
}
```

### Database Connection Error

```json
{
  "errors": [
    {
      "message": "Database connection failed",
      "extensions": {
        "code": "DATABASE_ERROR"
      }
    }
  ]
}
```

---

## Network Errors

These errors occur due to network connectivity issues.

### Connection Timeout

```json
{
  "errors": [
    {
      "message": "Request timeout",
      "extensions": {
        "code": "TIMEOUT"
      }
    }
  ]
}
```

### Rate Limiting

```json
{
  "errors": [
    {
      "message": "Rate limit exceeded. Please try again later.",
      "extensions": {
        "code": "RATE_LIMITED"
      }
    }
  ]
}
```

---

## Error Handling Best Practices

### 1. Always Handle Union Types in Mutations

```graphql
mutation CreateCustomerSafely {
  Customer {
    create(data: {
      fields: {
        name: "John Doe"
        email: "john@example.com"
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

### 2. Check for Errors in Every Response

```javascript
async function executeGraphQL(query, variables = {}) {
  try {
    const response = await fetch('/graphql', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': getApiKey()
      },
      body: JSON.stringify({ query, variables })
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const result = await response.json();

    // Check for GraphQL errors
    if (result.errors) {
      handleGraphQLErrors(result.errors);
      return null;
    }

    return result.data;
  } catch (error) {
    handleNetworkError(error);
    return null;
  }
}
```

### 3. Implement Comprehensive Error Handling

```javascript
function handleGraphQLErrors(errors) {
  errors.forEach(error => {
    const code = error.extensions?.code;
    
    switch (code) {
      case 'UNAUTHENTICATED':
        handleAuthenticationError(error);
        break;
      case 'FORBIDDEN':
        handleAuthorizationError(error);
        break;
      case 'GRAPHQL_VALIDATION_FAILED':
        handleValidationError(error);
        break;
      case 'RATE_LIMITED':
        handleRateLimitError(error);
        break;
      case 'INTERNAL_SERVER_ERROR':
        handleServerError(error);
        break;
      default:
        handleGenericError(error);
    }
  });
}
```

### 4. Provide User-Friendly Error Messages

```javascript
function getErrorMessage(error) {
  const code = error.extensions?.code;
  
  switch (code) {
    case 'UNAUTHENTICATED':
      return 'Please log in to access this resource.';
    case 'FORBIDDEN':
      return 'You don\'t have permission to perform this action.';
    case 'RATE_LIMITED':
      return 'Too many requests. Please try again in a few minutes.';
    case 'INTERNAL_SERVER_ERROR':
      return 'Something went wrong on our end. Please try again later.';
    default:
      return error.message || 'An unexpected error occurred.';
  }
}
```

### 5. Implement Retry Logic

```javascript
async function executeGraphQLWithRetry(query, variables = {}, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const result = await executeGraphQL(query, variables);
      return result;
    } catch (error) {
      if (attempt === maxRetries) {
        throw error;
      }
      
      // Wait before retrying (exponential backoff)
      const delay = Math.pow(2, attempt) * 1000;
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}
```

### 6. Log Errors for Debugging

```javascript
function logError(error, context = {}) {
  const errorInfo = {
    message: error.message,
    code: error.extensions?.code,
    locations: error.locations,
    timestamp: new Date().toISOString(),
    context
  };
  
  console.error('GraphQL Error:', errorInfo);
  
  // Send to error tracking service
  if (window.errorTracker) {
    window.errorTracker.captureException(error, errorInfo);
  }
}
```

### 7. Validate Input Before Sending

```javascript
function validateCustomerInput(input) {
  const errors = [];
  
  if (!input.name || input.name.trim() === '') {
    errors.push({ field: 'name', message: 'Name is required' });
  }
  
  if (!input.email || !isValidEmail(input.email)) {
    errors.push({ field: 'email', message: 'Valid email is required' });
  }
  
  if (input.annualRevenue && input.annualRevenue < 0) {
    errors.push({ field: 'annualRevenue', message: 'Revenue must be positive' });
  }
  
  return errors;
}

function isValidEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}
```

---

## Error Response Reference

### Standard Error Structure

```json
{
  "errors": [
    {
      "message": "Human-readable error message",
      "locations": [
        {
          "line": 2,
          "column": 5
        }
      ],
      "path": ["Customer", "create"],
      "extensions": {
        "code": "ERROR_CODE",
        "details": {
          "additional": "information"
        }
      }
    }
  ],
  "data": null
}
```

### Error Codes Reference

| Code | Description | Action |
|------|-------------|--------|
| `UNAUTHENTICATED` | Missing or invalid API key | Provide valid API key |
| `FORBIDDEN` | Insufficient permissions | Check user permissions |
| `GRAPHQL_VALIDATION_FAILED` | Invalid GraphQL query | Fix query syntax |
| `GRAPHQL_PARSE_FAILED` | Query parsing error | Check query structure |
| `RATE_LIMITED` | Too many requests | Implement rate limiting |
| `INTERNAL_SERVER_ERROR` | Server error | Retry or contact support |
| `DATABASE_ERROR` | Database issue | Retry or contact support |
| `TIMEOUT` | Request timeout | Retry with shorter timeout |
| `RECORD_NOT_FOUND` | Record doesn't exist | Check record ID |

---

<sub align="center">© 2025 InflowCRM &middot; GraphQL Error Handling</sub>