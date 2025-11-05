# API Conventions

> **Document Type**: Protocol  
> **Audience**: All API developers  
> **Last Updated**: November 5, 2024  
> **Status**: ✅ Approved & Enforced

---

## Overview

This document defines mandatory conventions for all Blog Engine APIs to ensure consistency, discoverability, and ease of use.

**All services MUST follow these conventions.**

---

## URL Structure

### Base URL Pattern
```
https://api.blogengine.com/{version}/{resource}
```

**Examples**:
- `https://api.blogengine.com/v1/posts`
- `https://api.blogengine.com/v1/users/123/posts`

### Rules
- ✅ Use **plural nouns** for resources (`/posts`, not `/post`)
- ✅ Use **kebab-case** for multi-word resources (`/user-profiles`)
- ✅ Use **path parameters** for resource IDs (`/posts/123`)
- ✅ Use **query parameters** for filtering/sorting (`?status=published&sort=created`)
- ❌ Never use verbs in URLs (`/getPosts` ❌, use `GET /posts` ✅)

---

## HTTP Methods

| Method | Purpose | Idempotent | Safe |
|--------|---------|------------|------|
| **GET** | Retrieve resource(s) | ✅ Yes | ✅ Yes |
| **POST** | Create new resource | ❌ No | ❌ No |
| **PUT** | Replace entire resource | ✅ Yes | ❌ No |
| **PATCH** | Partially update resource | ❌ No | ❌ No |
| **DELETE** | Remove resource | ✅ Yes | ❌ No |

### Examples

```http
GET    /posts           # List all posts
GET    /posts/123       # Get specific post
POST   /posts           # Create new post
PUT    /posts/123       # Replace entire post
PATCH  /posts/123       # Update specific fields
DELETE /posts/123       # Delete post
```

---

## Versioning

### Strategy: URL Path Versioning

```
/v1/posts    # Version 1
/v2/posts    # Version 2 (breaking changes)
```

### Rules
- Start with `/v1`
- Increment major version for **breaking changes** only
- Maintain **2 versions** simultaneously during migration (6 months overlap)
- Deprecation notices must be sent **3 months** before sunset

### Breaking Changes Include
- Removing fields from response
- Changing field types
- Changing authentication mechanism
- Renaming endpoints

### Non-Breaking Changes (same version)
- Adding new optional fields
- Adding new endpoints
- Adding new optional query parameters

---

## Request Format

### Content-Type
```http
Content-Type: application/json
```

All request bodies MUST be valid JSON.

### Request Body Example
```json
{
  "title": "My Blog Post",
  "content": "Post content here",
  "tags": ["nodejs", "api"],
  "status": "draft"
}
```

### Field Naming
- ✅ Use **camelCase** for JSON fields (`publishedAt`, not `published_at`)
- ✅ Be **consistent** across all endpoints
- ✅ Use **descriptive names** (`createdAt` not `ct`)

---

## Response Format

### Success Response Structure

**Single Resource**:
```json
{
  "id": 123,
  "title": "My Post",
  "content": "...",
  "author": {
    "id": 456,
    "name": "John Doe"
  },
  "createdAt": "2024-11-05T14:30:00Z"
}
```

**Collection** (with pagination):
```json
{
  "data": [
    { "id": 1, "title": "Post 1" },
    { "id": 2, "title": "Post 2" }
  ],
  "pagination": {
    "page": 1,
    "perPage": 20,
    "total": 156,
    "totalPages": 8
  }
}
```

### Error Response Structure

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request data",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ],
    "requestId": "abc-123-def"
  }
}
```

See [Error Handling](error-handling.md) for full error codes.

---

## HTTP Status Codes

### Success Codes
- **200 OK**: Request succeeded (GET, PATCH, DELETE)
- **201 Created**: Resource created successfully (POST)
- **204 No Content**: Success with no response body (DELETE)

### Client Error Codes
- **400 Bad Request**: Invalid request data
- **401 Unauthorized**: Missing or invalid authentication
- **403 Forbidden**: Authenticated but not authorized
- **404 Not Found**: Resource doesn't exist
- **409 Conflict**: Resource conflict (e.g., duplicate)
- **422 Unprocessable Entity**: Validation failed
- **429 Too Many Requests**: Rate limit exceeded

### Server Error Codes
- **500 Internal Server Error**: Unexpected server error
- **502 Bad Gateway**: Upstream service failure
- **503 Service Unavailable**: Service temporarily down

---

## Pagination

### Query Parameters
```
?page=1&limit=20
```

**Parameters**:
- `page`: Page number (1-indexed, default: 1)
- `limit`: Items per page (default: 20, max: 100)

### Response Format
```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "perPage": 20,
    "total": 156,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": false
  }
}
```

### Cursor-Based Pagination (for large datasets)
```
?cursor=eyJpZCI6MTIzfQ&limit=20
```

Response includes `nextCursor` and `prevCursor` in pagination object.

---

## Filtering & Sorting

### Filtering
```
GET /posts?status=published&author=123&tag=nodejs
```

**Rules**:
- Use query parameters for filters
- Support multiple values with comma: `?status=published,draft`
- Use operators for ranges: `?views_gte=100&views_lt=1000`

**Common Operators**:
- `_eq`: Equal (default if no operator)
- `_ne`: Not equal
- `_gt`: Greater than
- `_gte`: Greater than or equal
- `_lt`: Less than
- `_lte`: Less than or equal
- `_in`: In list (comma-separated)

### Sorting
```
GET /posts?sort=createdAt          # Ascending
GET /posts?sort=-createdAt         # Descending (prefix with -)
GET /posts?sort=status,-createdAt  # Multiple fields
```

### Full Query Example
```
GET /posts?status=published&author=123&sort=-createdAt&page=2&limit=20
```

---

## Field Selection (Sparse Fieldsets)

Request only needed fields to reduce payload size:

```
GET /posts?fields=id,title,author.name
```

Response:
```json
{
  "data": [
    {
      "id": 1,
      "title": "Post Title",
      "author": {
        "name": "John Doe"
      }
    }
  ]
}
```

---

## Headers

### Required Request Headers
```http
Authorization: Bearer <token>      # For authenticated requests
Content-Type: application/json     # For POST/PUT/PATCH
Accept: application/json           # Response format
```

### Standard Response Headers
```http
Content-Type: application/json
X-Request-ID: abc-123-def          # For request tracing
X-RateLimit-Limit: 1000            # Rate limit max
X-RateLimit-Remaining: 999         # Requests remaining
X-RateLimit-Reset: 1699200000      # Reset timestamp
```

---

## Rate Limiting

### Limits
- **Authenticated users**: 1,000 requests/hour
- **Unauthenticated**: 100 requests/hour
- **Admin users**: 5,000 requests/hour

### Response When Exceeded
```http
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1699203600
Retry-After: 3600

{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Try again in 3600 seconds.",
    "retryAfter": 3600
  }
}
```

---

## CORS Configuration

### Allowed Origins
- `https://blogengine.com`
- `https://*.blogengine.com`
- `http://localhost:*` (development only)

### Allowed Methods
```
GET, POST, PUT, PATCH, DELETE, OPTIONS
```

### Allowed Headers
```
Authorization, Content-Type, Accept, X-Request-ID
```

---

## Timestamps

### Format
All timestamps MUST use **ISO 8601** format with UTC timezone:

```json
{
  "createdAt": "2024-11-05T14:30:00Z",
  "publishedAt": "2024-11-05T15:00:00Z"
}
```

### Field Naming
- `createdAt`: Resource creation time
- `updatedAt`: Last modification time
- `publishedAt`: Publication time
- `deletedAt`: Soft deletion time

---

## Null vs Omission

- **Null values**: Field exists but has no value
- **Omitted fields**: Field not included in response

```json
{
  "title": "Post Title",
  "excerpt": null,          // Explicitly null
  // publishedAt omitted (post is draft)
}
```

---

## Idempotency

### POST Requests (Idempotency Keys)
For critical operations, support idempotency keys:

```http
POST /payments
Idempotency-Key: unique-key-123

{
  "amount": 100,
  "currency": "USD"
}
```

Same key within 24 hours returns same result (201 first time, 200 subsequent).

---

## Deprecation

### Headers
```http
Sunset: Sat, 01 Jun 2025 00:00:00 GMT
Deprecation: Sat, 01 Mar 2025 00:00:00 GMT
Link: <https://api.blogengine.com/v2/posts>; rel="successor-version"
```

### Response Warning
```json
{
  "data": {...},
  "warnings": [
    {
      "code": "DEPRECATED_ENDPOINT",
      "message": "This endpoint will be removed on 2025-06-01. Use /v2/posts instead.",
      "sunsetDate": "2025-06-01"
    }
  ]
}
```

---

## Examples

### Complete Request/Response

**Request**:
```http
POST /v1/posts HTTP/1.1
Host: api.blogengine.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json
Accept: application/json

{
  "title": "Getting Started with Node.js",
  "content": "This is a comprehensive guide...",
  "tags": ["nodejs", "javascript", "tutorial"],
  "status": "draft"
}
```

**Response**:
```http
HTTP/1.1 201 Created
Content-Type: application/json
X-Request-ID: req_abc123def456
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
Location: /v1/posts/157

{
  "id": 157,
  "title": "Getting Started with Node.js",
  "slug": "getting-started-with-nodejs",
  "content": "This is a comprehensive guide...",
  "excerpt": "This is a comprehensive guide to...",
  "author": {
    "id": 123,
    "name": "John Doe",
    "avatarUrl": "https://cdn.blogengine.com/avatars/john.jpg"
  },
  "tags": [
    { "id": 1, "name": "Node.js", "slug": "nodejs" },
    { "id": 2, "name": "JavaScript", "slug": "javascript" },
    { "id": 3, "name": "Tutorial", "slug": "tutorial" }
  ],
  "status": "draft",
  "viewCount": 0,
  "likeCount": 0,
  "commentCount": 0,
  "createdAt": "2024-11-05T14:30:00Z",
  "updatedAt": "2024-11-05T14:30:00Z"
}
```

---

## Validation

All APIs must validate:
- ✅ Required fields present
- ✅ Field types correct
- ✅ String lengths within limits
- ✅ Enum values are valid
- ✅ Foreign keys exist (referential integrity)

Return **422 Unprocessable Entity** with detailed errors.

---

## Related Documentation

- [Error Handling Standards](error-handling.md)
- [Event Schema](event-schema.md)
- [Logging Standards](logging-standards.md)
- [API Security](../security/api-security.md)

---

**Maintained By**: API Standards Committee  
**Review Frequency**: Quarterly  
**Next Review**: February 1, 2025
