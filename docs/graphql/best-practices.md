# InflowCRM GraphQL API – Best Practices

This document provides comprehensive best practices for using the InflowCRM GraphQL API effectively, covering performance optimization, security, error handling, and development workflows.

---

## Table of Contents

- [Query Design](#query-design)
- [Performance Optimization](#performance-optimization)
- [Security Best Practices](#security-best-practices)
- [Error Handling](#error-handling)
- [Development Workflow](#development-workflow)
- [Production Considerations](#production-considerations)

---

## Query Design

### 1. Request Only Needed Fields

GraphQL's power lies in fetching exactly what you need. Avoid over-fetching data.

**❌ Bad: Over-fetching**
```graphql
query {
  Customer {
    getMany(perPage: 10, page: 1) {
      id
      createdTime
      modifiedTime
      fields {
        name
        email
        phone
        companyName
        address
        city
        state
        zipCode
        country
        website
        industry
        annualRevenue
        employeeCount
        description
        notes
        # ... many more fields
      }
    }
  }
}
```

**✅ Good: Selective fetching**
```graphql
query {
  Customer {
    getMany(perPage: 10, page: 1) {
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

### 2. Use Fragments for Reusable Field Sets

Create reusable fragments for common field combinations.

**✅ Good: Using fragments**
```graphql
fragment CustomerBasicInfo on Customer {
  id
  createdTime
  fields {
    name
    email
    phone
    companyName
  }
}

fragment CustomerDetailInfo on Customer {
  ...CustomerBasicInfo
  modifiedTime
  fields {
    address
    city
    state
    industry
    annualRevenue
    employeeCount
  }
}

query GetCustomerList {
  Customer {
    getMany(perPage: 20, page: 1) {
      ...CustomerBasicInfo
    }
  }
}

query GetCustomerDetails($id: ID!) {
  Customer {
    getOneById(id: $id) {
      ...CustomerDetailInfo
      contacts {
        id
        fields {
          firstName
          lastName
          email
        }
      }
    }
  }
}
```

### 3. Use Descriptive Operation Names

Always name your operations clearly to improve debugging and monitoring.

**❌ Bad: Unnamed operations**
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
}
```

**✅ Good: Named operations**
```graphql
query GetCustomerListForDashboard {
  Customer {
    getMany(perPage: 10, page: 1) {
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

### 4. Leverage Variables for Dynamic Queries

Use variables instead of string interpolation for security and reusability.

**❌ Bad: String interpolation**
```javascript
const query = `
  query {
    Customer {
      getOneById(id: "${customerId}") {
        id
        fields {
          name
        }
      }
    }
  }
`;
```

**✅ Good: Using variables**
```javascript
const query = `
  query GetCustomerById($id: ID!) {
    Customer {
      getOneById(id: $id) {
        id
        fields {
          name
        }
      }
    }
  }
`;

const variables = { id: customerId };
```

---

## Performance Optimization

### 1. Implement Pagination

Always use pagination for list queries to avoid performance issues.

**✅ Good: Paginated queries**
```graphql
query GetCustomersPaginated($page: Int!, $perPage: Int!) {
  Customer {
    getMany(page: $page, perPage: $perPage) {
      id
      fields {
        name
        email
      }
    }
  }
}
```

### 2. Optimize Filtering

Use specific filters to reduce the dataset size early.

**❌ Bad: Broad filtering**
```graphql
query {
  Customer {
    getMany(perPage: 100, page: 1, filtration: {
      fields: {
        name: { contains: "a" }
      }
    }) {
      id
      fields {
        name
        email
      }
    }
  }
}
```

**✅ Good: Specific filtering**
```graphql
query GetActiveCustomersInTech {
  Customer {
    getMany(perPage: 20, page: 1, filtration: {
      fields: {
        status: { equal: "ACTIVE" }
        industry: { in: ["Technology", "Software"] }
        annualRevenue: { greaterOrEqual: 1000000 }
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

### 3. Batch Related Data

Fetch related data in a single query rather than multiple requests.

**❌ Bad: Multiple requests**
```javascript
// First request
const customer = await getCustomerById(id);

// Second request
const contacts = await getContactsByCustomerId(id);

// Third request
const opportunities = await getOpportunitiesByCustomerId(id);
```

**✅ Good: Single request with relations**
```graphql
query GetCustomerWithRelations($id: ID!) {
  Customer {
    getOneById(id: $id) {
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
    }
  }
}
```

### 4. Use Appropriate Page Sizes

Choose optimal page sizes based on your use case.

```javascript
// For list views
const LIST_PAGE_SIZE = 20;

// For dropdown/select components
const DROPDOWN_PAGE_SIZE = 100;

// For data exports
const EXPORT_PAGE_SIZE = 500;
```

### 5. Implement Client-Side Caching

Cache frequently accessed data to reduce server requests.

```javascript
// Using a simple in-memory cache
const cache = new Map();

async function getCachedCustomer(id) {
  const cacheKey = `customer:${id}`;
  
  if (cache.has(cacheKey)) {
    return cache.get(cacheKey);
  }
  
  const customer = await fetchCustomerById(id);
  cache.set(cacheKey, customer);
  
  // Cache for 5 minutes
  setTimeout(() => cache.delete(cacheKey), 5 * 60 * 1000);
  
  return customer;
}
```

---

## Security Best Practices

### 1. Secure API Key Management

Never expose API keys in client-side code or public repositories.

**❌ Bad: Exposed API key**
```javascript
// Don't do this in frontend code
const API_KEY = 'your-secret-api-key';

fetch('/graphql', {
  headers: {
    'x-api-key': API_KEY
  }
});
```

**✅ Good: Server-side proxy**
```javascript
// Frontend makes requests to your backend
fetch('/api/graphql-proxy', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${userToken}`
  },
  body: JSON.stringify({ query, variables })
});

// Backend proxy adds API key
app.post('/api/graphql-proxy', authenticate, async (req, res) => {
  const response = await fetch('https://srv.inflowcrm.pl/graphql', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': process.env.INFLOWCRM_API_KEY
    },
    body: JSON.stringify(req.body)
  });
  
  const result = await response.json();
  res.json(result);
});
```

### 2. Input Validation

Always validate input data before sending to the API.

**✅ Good: Input validation**
```javascript
function validateCustomerInput(input) {
  const errors = [];
  
  // Required fields
  if (!input.name?.trim()) {
    errors.push({ field: 'name', message: 'Name is required' });
  }
  
  // Email validation
  if (!input.email || !isValidEmail(input.email)) {
    errors.push({ field: 'email', message: 'Valid email is required' });
  }
  
  // Business rules
  if (input.annualRevenue && input.annualRevenue < 0) {
    errors.push({ field: 'annualRevenue', message: 'Revenue must be positive' });
  }
  
  return errors;
}

function isValidEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}
```

### 3. Rate Limiting

Implement client-side rate limiting to avoid hitting API limits.

```javascript
class RateLimiter {
  constructor(maxRequests = 300, windowMs = 5 * 60 * 1000) {
    this.maxRequests = maxRequests;
    this.windowMs = windowMs;
    this.requests = [];
  }
  
  async checkLimit() {
    const now = Date.now();
    
    // Remove old requests outside the window
    this.requests = this.requests.filter(time => now - time < this.windowMs);
    
    if (this.requests.length >= this.maxRequests) {
      const oldestRequest = Math.min(...this.requests);
      const waitTime = this.windowMs - (now - oldestRequest);
      
      throw new Error(`Rate limit exceeded. Wait ${waitTime}ms before next request.`);
    }
    
    this.requests.push(now);
  }
}

const rateLimiter = new RateLimiter();

async function makeGraphQLRequest(query, variables) {
  await rateLimiter.checkLimit();
  
  return fetch('/graphql', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': getApiKey()
    },
    body: JSON.stringify({ query, variables })
  });
}
```

### 4. Sanitize User Input

Sanitize user input to prevent injection attacks.

```javascript
function sanitizeInput(input) {
  if (typeof input !== 'string') return input;
  
  return input
    .trim()
    .replace(/[<>]/g, '') // Remove potential HTML tags
    .slice(0, 1000); // Limit length
}

function sanitizeCustomerData(data) {
  return {
    name: sanitizeInput(data.name),
    email: sanitizeInput(data.email),
    phone: sanitizeInput(data.phone),
    companyName: sanitizeInput(data.companyName),
    description: sanitizeInput(data.description)
  };
}
```

---

## Error Handling

### 1. Comprehensive Error Handling

Handle all types of errors that can occur.

```javascript
async function executeGraphQLQuery(query, variables = {}) {
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
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    
    const result = await response.json();
    
    // Handle GraphQL errors
    if (result.errors) {
      throw new GraphQLError(result.errors);
    }
    
    return result.data;
  } catch (error) {
    if (error instanceof GraphQLError) {
      handleGraphQLError(error);
    } else {
      handleNetworkError(error);
    }
    throw error;
  }
}

class GraphQLError extends Error {
  constructor(errors) {
    super('GraphQL Error');
    this.graphQLErrors = errors;
  }
}

function handleGraphQLError(error) {
  error.graphQLErrors.forEach(err => {
    console.error('GraphQL Error:', err.message);
    
    if (err.extensions?.code === 'UNAUTHENTICATED') {
      redirectToLogin();
    } else if (err.extensions?.code === 'RATE_LIMITED') {
      showRateLimitMessage();
    }
  });
}
```

### 2. Graceful Degradation

Implement fallbacks when API calls fail.

```javascript
async function getCustomerWithFallback(id) {
  try {
    return await getCustomerById(id);
  } catch (error) {
    console.error('Failed to fetch customer:', error);
    
    // Return cached data if available
    const cached = getCachedCustomer(id);
    if (cached) {
      return { ...cached, isStale: true };
    }
    
    // Return minimal data structure
    return {
      id,
      fields: {
        name: 'Unknown Customer',
        email: '',
        phone: ''
      },
      isError: true
    };
  }
}
```

### 3. User-Friendly Error Messages

Provide clear, actionable error messages to users.

```javascript
function getErrorMessage(error) {
  if (error.graphQLErrors) {
    const firstError = error.graphQLErrors[0];
    
    switch (firstError.extensions?.code) {
      case 'UNAUTHENTICATED':
        return 'Your session has expired. Please log in again.';
      case 'FORBIDDEN':
        return 'You don\'t have permission to perform this action.';
      case 'RATE_LIMITED':
        return 'Too many requests. Please wait a moment and try again.';
      case 'VALIDATION_FAILED':
        return 'Please check your input and try again.';
      default:
        return 'An error occurred. Please try again.';
    }
  }
  
  return 'Network error. Please check your connection and try again.';
}
```

---

## Development Workflow

### 1. Schema Introspection

Use introspection to understand your schema during development.

```javascript
const introspectionQuery = `
  query IntrospectionQuery {
    __schema {
      queryType {
        fields {
          name
          description
          type {
            name
            kind
          }
        }
      }
      mutationType {
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
  }
`;

async function exploreSchema() {
  const result = await executeGraphQLQuery(introspectionQuery);
  console.log('Available queries:', result.__schema.queryType.fields);
  console.log('Available mutations:', result.__schema.mutationType.fields);
}
```

### 2. Development vs Production Configurations

Use different configurations for development and production.

```javascript
const config = {
  development: {
    endpoint: 'http://localhost:3000/graphql',
    enableLogging: true,
    enableIntrospection: true,
    cacheTimeout: 60000 // 1 minute
  },
  production: {
    endpoint: 'https://srv.inflowcrm.pl/graphql',
    enableLogging: false,
    enableIntrospection: false,
    cacheTimeout: 300000 // 5 minutes
  }
};

const env = process.env.NODE_ENV || 'development';
const currentConfig = config[env];
```

### 3. Testing Strategies

Implement comprehensive testing for GraphQL operations.

```javascript
// Unit tests for GraphQL operations
describe('Customer GraphQL Operations', () => {
  test('should fetch customer by ID', async () => {
    const mockCustomer = {
      id: '123',
      fields: {
        name: 'Test Customer',
        email: 'test@example.com'
      }
    };
    
    mockGraphQLResponse(mockCustomer);
    
    const result = await getCustomerById('123');
    expect(result).toEqual(mockCustomer);
  });
  
  test('should handle customer not found', async () => {
    mockGraphQLResponse(null);
    
    const result = await getCustomerById('nonexistent');
    expect(result).toBeNull();
  });
  
  test('should handle validation errors', async () => {
    const mockError = {
      validationErrors: [
        {
          fieldName: 'email',
          errorType: 'REQUIRED',
          description: 'Email is required'
        }
      ]
    };
    
    mockGraphQLResponse(mockError);
    
    const result = await createCustomer({ name: 'Test' });
    expect(result.validationErrors).toBeDefined();
  });
});
```

### 4. Code Organization

Organize GraphQL-related code in a maintainable structure.

```
src/
├── graphql/
│   ├── queries/
│   │   ├── customer.js
│   │   ├── contact.js
│   │   └── opportunity.js
│   ├── mutations/
│   │   ├── customer.js
│   │   ├── contact.js
│   │   └── opportunity.js
│   ├── fragments/
│   │   ├── customer.js
│   │   └── contact.js
│   └── client.js
├── utils/
│   ├── validation.js
│   ├── errorHandling.js
│   └── cache.js
└── types/
    └── customer.ts
```

---

## Production Considerations

### 1. Monitoring and Logging

Implement comprehensive monitoring for GraphQL operations.

```javascript
function logGraphQLOperation(operation, variables, result, duration) {
  const logData = {
    operation: operation.operationName,
    variables,
    duration,
    timestamp: new Date().toISOString(),
    success: !result.errors,
    errors: result.errors?.map(e => ({
      message: e.message,
      code: e.extensions?.code
    }))
  };
  
  if (result.errors) {
    console.error('GraphQL Error:', logData);
  } else {
    console.log('GraphQL Success:', logData);
  }
  
  // Send to monitoring service
  sendToMonitoring(logData);
}
```

### 2. Performance Monitoring

Track query performance and identify bottlenecks.

```javascript
class PerformanceMonitor {
  constructor() {
    this.metrics = new Map();
  }
  
  startOperation(operationName) {
    const startTime = performance.now();
    return {
      operationName,
      startTime,
      end: () => {
        const duration = performance.now() - startTime;
        this.recordMetric(operationName, duration);
        return duration;
      }
    };
  }
  
  recordMetric(operationName, duration) {
    if (!this.metrics.has(operationName)) {
      this.metrics.set(operationName, []);
    }
    
    this.metrics.get(operationName).push(duration);
    
    // Keep only last 100 measurements
    const measurements = this.metrics.get(operationName);
    if (measurements.length > 100) {
      measurements.shift();
    }
  }
  
  getStats(operationName) {
    const measurements = this.metrics.get(operationName) || [];
    if (measurements.length === 0) return null;
    
    const sorted = [...measurements].sort((a, b) => a - b);
    return {
      count: measurements.length,
      avg: measurements.reduce((a, b) => a + b, 0) / measurements.length,
      min: sorted[0],
      max: sorted[sorted.length - 1],
      p95: sorted[Math.floor(sorted.length * 0.95)]
    };
  }
}
```

### 3. Error Reporting

Implement proper error reporting for production issues.

```javascript
function setupErrorReporting() {
  window.addEventListener('unhandledrejection', (event) => {
    if (event.reason instanceof GraphQLError) {
      reportError({
        type: 'GraphQL Error',
        errors: event.reason.graphQLErrors,
        url: window.location.href,
        userAgent: navigator.userAgent,
        timestamp: new Date().toISOString()
      });
    }
  });
}

function reportError(errorData) {
  // Send to error tracking service
  fetch('/api/errors', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(errorData)
  }).catch(err => {
    console.error('Failed to report error:', err);
  });
}
```

### 4. Deployment Checklist

Before deploying to production:

- [ ] API keys are properly secured
- [ ] Rate limiting is implemented
- [ ] Error handling is comprehensive
- [ ] Logging and monitoring are in place
- [ ] Input validation is implemented
- [ ] Caching strategies are optimized
- [ ] Performance metrics are being collected
- [ ] Fallback mechanisms are in place
- [ ] Security headers are configured
- [ ] HTTPS is enforced

---

## Summary

Following these best practices will help you:

1. **Build efficient queries** that fetch only necessary data
2. **Optimize performance** through caching and pagination
3. **Implement robust security** measures
4. **Handle errors gracefully** with proper fallbacks
5. **Maintain clean code** that's easy to debug and extend
6. **Monitor and optimize** production performance

Remember that GraphQL is a powerful tool, but with great power comes great responsibility. Always consider the impact of your queries on server performance and user experience.

---

<sub align="center">© 2025 InflowCRM &middot; GraphQL Best Practices</sub>