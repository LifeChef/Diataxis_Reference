# Error Handling Standards

> **Document Type**: Protocol  
> **Audience**: All API developers  
> **Last Updated**: November 5, 2024  
> **Status**: ✅ Approved & Enforced

---

## Overview

This document defines standard error handling patterns for all Blog Engine APIs to ensure consistent, debuggable, and user-friendly error responses.

**All services MUST use these error formats and codes.**

---

## Error Response Structure

### Standard Envelope

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "details": [...],
    "requestId": "req_abc123def456",
    "timestamp": "2024-11-05T14:30:00Z",
    "path": "/api/v1/posts"
  }
}
```

### Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `code` | string | ✅ | Machine-readable error code (UPPER_SNAKE_CASE) |
| `message` | string | ✅ | Human-readable error description |
| `details` | array | ❌ | Additional error details (validation errors) |
| `requestId` | string | ✅ | Unique request ID for tracing |
| `timestamp` | string | ✅ | ISO 8601 timestamp when error occurred |
| `path` | string | ✅ | API path that generated the error |

---

## Error Codes

### Client Errors (4xx)

#### 400 Bad Request

**BAD_REQUEST**
```json
{
  "error": {
    "code": "BAD_REQUEST",
    "message": "Invalid request format",
    "requestId": "req_123"
  }
}
```

**INVALID_JSON**
```json
{
  "error": {
    "code": "INVALID_JSON",
    "message": "Request body is not valid JSON",
    "details": [
      {
        "position": 45,
        "error": "Unexpected token }"
      }
    ],
    "requestId": "req_123"
  }
}
```

---

#### 401 Unauthorized

**MISSING_AUTHORIZATION**
```json
{
  "error": {
    "code": "MISSING_AUTHORIZATION",
    "message": "Authorization header is required",
    "requestId": "req_123"
  }
}
```

**INVALID_TOKEN**
```json
{
  "error": {
    "code": "INVALID_TOKEN",
    "message": "Invalid or malformed authentication token",
    "requestId": "req_123"
  }
}
```

**TOKEN_EXPIRED**
```json
{
  "error": {
    "code": "TOKEN_EXPIRED",
    "message": "Authentication token has expired",
    "details": [
      {
        "expiredAt": "2024-11-05T14:00:00Z",
        "currentTime": "2024-11-05T14:30:00Z"
      }
    ],
    "requestId": "req_123"
  }
}
```

---

#### 403 Forbidden

**FORBIDDEN**
```json
{
  "error": {
    "code": "FORBIDDEN",
    "message": "You do not have permission to access this resource",
    "details": [
      {
        "resource": "post",
        "resourceId": 123,
        "requiredPermission": "post:delete",
        "userPermissions": ["post:read", "post:create"]
      }
    ],
    "requestId": "req_123"
  }
}
```

**INSUFFICIENT_PERMISSIONS**
```json
{
  "error": {
    "code": "INSUFFICIENT_PERMISSIONS",
    "message": "User role 'viewer' cannot perform this action",
    "requestId": "req_123"
  }
}
```

---

#### 404 Not Found

**RESOURCE_NOT_FOUND**
```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "The requested resource was not found",
    "details": [
      {
        "resource": "post",
        "resourceId": 999
      }
    ],
    "requestId": "req_123"
  }
}
```

**ENDPOINT_NOT_FOUND**
```json
{
  "error": {
    "code": "ENDPOINT_NOT_FOUND",
    "message": "The requested endpoint does not exist",
    "path": "/api/v1/invalid-endpoint",
    "requestId": "req_123"
  }
}
```

---

#### 409 Conflict

**RESOURCE_CONFLICT**
```json
{
  "error": {
    "code": "RESOURCE_CONFLICT",
    "message": "Resource already exists",
    "details": [
      {
        "field": "email",
        "value": "user@example.com",
        "existingResourceId": 456
      }
    ],
    "requestId": "req_123"
  }
}
```

**DUPLICATE_ENTRY**
```json
{
  "error": {
    "code": "DUPLICATE_ENTRY",
    "message": "A post with this slug already exists",
    "details": [
      {
        "field": "slug",
        "value": "my-blog-post",
        "conflictingResourceId": 789
      }
    ],
    "requestId": "req_123"
  }
}
```

---

#### 422 Unprocessable Entity

**VALIDATION_ERROR**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "title",
        "message": "Title is required",
        "rejectedValue": null
      },
      {
        "field": "content",
        "message": "Content must be at least 100 characters",
        "rejectedValue": "Too short",
        "constraint": {
          "minLength": 100,
          "actualLength": 9
        }
      },
      {
        "field": "email",
        "message": "Invalid email format",
        "rejectedValue": "not-an-email",
        "pattern": "^[^@]+@[^@]+\\.[^@]+$"
      }
    ],
    "requestId": "req_123"
  }
}
```

**INVALID_FIELD_VALUE**
```json
{
  "error": {
    "code": "INVALID_FIELD_VALUE",
    "message": "Field value is invalid",
    "details": [
      {
        "field": "status",
        "rejectedValue": "invalid_status",
        "allowedValues": ["draft", "published", "archived"]
      }
    ],
    "requestId": "req_123"
  }
}
```

---

#### 429 Too Many Requests

**RATE_LIMIT_EXCEEDED**
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Try again later.",
    "details": [
      {
        "limit": 1000,
        "remaining": 0,
        "resetAt": "2024-11-05T15:00:00Z",
        "retryAfter": 3600
      }
    ],
    "requestId": "req_123"
  }
}
```

---

### Server Errors (5xx)

#### 500 Internal Server Error

**INTERNAL_SERVER_ERROR**
```json
{
  "error": {
    "code": "INTERNAL_SERVER_ERROR",
    "message": "An unexpected error occurred. Please try again later.",
    "requestId": "req_123",
    "timestamp": "2024-11-05T14:30:00Z"
  }
}
```

**Note**: Never expose internal error details (stack traces, DB errors) to clients.

---

#### 502 Bad Gateway

**UPSTREAM_SERVICE_ERROR**
```json
{
  "error": {
    "code": "UPSTREAM_SERVICE_ERROR",
    "message": "Upstream service is unavailable",
    "details": [
      {
        "service": "database",
        "status": "unavailable"
      }
    ],
    "requestId": "req_123"
  }
}
```

---

#### 503 Service Unavailable

**SERVICE_UNAVAILABLE**
```json
{
  "error": {
    "code": "SERVICE_UNAVAILABLE",
    "message": "Service is temporarily unavailable. Please try again later.",
    "details": [
      {
        "retryAfter": 60,
        "estimatedDowntime": "5 minutes"
      }
    ],
    "requestId": "req_123"
  }
}
```

**MAINTENANCE_MODE**
```json
{
  "error": {
    "code": "MAINTENANCE_MODE",
    "message": "Service is under maintenance",
    "details": [
      {
        "plannedEnd": "2024-11-05T16:00:00Z",
        "message": "Scheduled maintenance for database upgrade"
      }
    ],
    "requestId": "req_123"
  }
}
```

---

## Validation Error Details

### Field-Level Validation

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "location": "body",
        "message": "Invalid email format",
        "rejectedValue": "invalid-email",
        "constraint": {
          "type": "format",
          "pattern": "email"
        }
      },
      {
        "field": "age",
        "location": "body",
        "message": "Age must be at least 18",
        "rejectedValue": 15,
        "constraint": {
          "type": "minimum",
          "value": 18
        }
      },
      {
        "field": "tags",
        "location": "body",
        "message": "Maximum 5 tags allowed",
        "rejectedValue": ["tag1", "tag2", "tag3", "tag4", "tag5", "tag6"],
        "constraint": {
          "type": "maxItems",
          "value": 5
        }
      }
    ],
    "requestId": "req_123"
  }
}
```

### Constraint Types

| Type | Description | Example |
|------|-------------|---------|
| `required` | Field is required | `{"type": "required"}` |
| `minLength` | Minimum string length | `{"type": "minLength", "value": 8}` |
| `maxLength` | Maximum string length | `{"type": "maxLength", "value": 200}` |
| `minimum` | Minimum number value | `{"type": "minimum", "value": 1}` |
| `maximum` | Maximum number value | `{"type": "maximum", "value": 100}` |
| `pattern` | Regex pattern | `{"type": "pattern", "value": "^[A-Z]"}` |
| `format` | Specific format | `{"type": "format", "value": "email"}` |
| `enum` | Allowed values | `{"type": "enum", "values": ["A", "B"]}` |
| `uniqueItems` | Array items must be unique | `{"type": "uniqueItems"}` |

---

## HTTP Response Headers

### Required Headers

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json
X-Request-ID: req_abc123def456
```

### Optional Headers

```http
Retry-After: 3600                           # For 429, 503
X-RateLimit-Limit: 1000                     # For 429
X-RateLimit-Remaining: 0                    # For 429
X-RateLimit-Reset: 1699203600               # For 429
```

---

## Error Logging

### Server-Side Logging

Always log errors with full context (never expose to client):

```javascript
logger.error('Database connection failed', {
  error: err.message,
  stack: err.stack,
  requestId: req.id,
  userId: req.user?.id,
  path: req.path,
  method: req.method,
  query: req.query,
  body: req.body,
  timestamp: new Date().toISOString()
});
```

### What NOT to Expose

❌ Stack traces  
❌ Database query errors  
❌ File paths  
❌ Internal service names  
❌ Credentials or secrets  
❌ User PII in logs

---

## Error Handling Best Practices

### DO ✅

- **Use specific error codes** for different scenarios
- **Include requestId** in all error responses
- **Provide actionable messages** ("Email is required" not "Invalid input")
- **Return correct HTTP status codes**
- **Log errors server-side** with full context
- **Validate input** before processing
- **Use structured error format** consistently

### DON'T ❌

- **Expose internal errors** to clients
- **Use generic error messages** ("Something went wrong")
- **Include sensitive data** in error responses
- **Return 200 OK** for errors
- **Ignore errors silently**
- **Use error codes inconsistently**

---

## Implementation Examples

### Express.js Error Handler

```javascript
// Error handling middleware
app.use((err, req, res, next) => {
  // Log error server-side
  logger.error('Request error', {
    error: err.message,
    stack: err.stack,
    requestId: req.id,
    path: req.path
  });

  // Determine status code
  const statusCode = err.statusCode || 500;

  // Build error response
  const errorResponse = {
    error: {
      code: err.code || 'INTERNAL_SERVER_ERROR',
      message: err.message || 'An unexpected error occurred',
      requestId: req.id,
      timestamp: new Date().toISOString(),
      path: req.path
    }
  };

  // Add details if available
  if (err.details) {
    errorResponse.error.details = err.details;
  }

  // Never expose internal errors in production
  if (process.env.NODE_ENV === 'production' && statusCode === 500) {
    errorResponse.error.message = 'An unexpected error occurred';
  }

  res.status(statusCode).json(errorResponse);
});
```

### Custom Error Classes

```javascript
class ValidationError extends Error {
  constructor(details) {
    super('Validation failed');
    this.name = 'ValidationError';
    this.code = 'VALIDATION_ERROR';
    this.statusCode = 422;
    this.details = details;
  }
}

class NotFoundError extends Error {
  constructor(resource, resourceId) {
    super(`${resource} not found`);
    this.name = 'NotFoundError';
    this.code = 'RESOURCE_NOT_FOUND';
    this.statusCode = 404;
    this.details = [{ resource, resourceId }];
  }
}

// Usage
throw new ValidationError([
  { field: 'email', message: 'Invalid email format' }
]);

throw new NotFoundError('post', 123);
```

---

## Testing Error Responses

### Example Tests

```javascript
describe('Error Handling', () => {
  it('should return 400 for invalid JSON', async () => {
    const response = await request(app)
      .post('/api/posts')
      .send('invalid json')
      .expect(400);

    expect(response.body.error.code).toBe('INVALID_JSON');
    expect(response.body.error.requestId).toBeDefined();
  });

  it('should return 422 for validation errors', async () => {
    const response = await request(app)
      .post('/api/posts')
      .send({ title: '' }) // Empty title
      .expect(422);

    expect(response.body.error.code).toBe('VALIDATION_ERROR');
    expect(response.body.error.details).toHaveLength(1);
    expect(response.body.error.details[0].field).toBe('title');
  });

  it('should return 404 for non-existent resource', async () => {
    const response = await request(app)
      .get('/api/posts/99999')
      .expect(404);

    expect(response.body.error.code).toBe('RESOURCE_NOT_FOUND');
  });
});
```

---

## Error Code Registry

Maintain centralized error code registry:

```javascript
// errors/codes.js
const ERROR_CODES = {
  // Auth
  MISSING_AUTHORIZATION: { status: 401, message: 'Authorization required' },
  INVALID_TOKEN: { status: 401, message: 'Invalid token' },
  TOKEN_EXPIRED: { status: 401, message: 'Token expired' },
  
  // Validation
  VALIDATION_ERROR: { status: 422, message: 'Validation failed' },
  INVALID_FIELD_VALUE: { status: 422, message: 'Invalid field value' },
  
  // Resources
  RESOURCE_NOT_FOUND: { status: 404, message: 'Resource not found' },
  DUPLICATE_ENTRY: { status: 409, message: 'Duplicate entry' },
  
  // Server
  INTERNAL_SERVER_ERROR: { status: 500, message: 'Internal error' }
};

module.exports = ERROR_CODES;
```

---

## Related Documentation

- [API Conventions](api-conventions.md)
- [Logging Standards](logging-standards.md)
- [API Security](../security/api-security.md)

---

**Maintained By**: API Standards Committee  
**Review Frequency**: Quarterly  
**Next Review**: February 1, 2025
