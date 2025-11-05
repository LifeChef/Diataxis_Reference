# API Endpoints Reference

> **Document Type**: Reference (Information-Oriented)  
> **Category**: API  
> **Version**: 1.0  
> **Last Updated**: November 5, 2024

## Overview

Complete reference for all Blog Engine REST API endpoints.

**Base URL**: `https://api.blogengine.com/v1`  
**Authentication**: Bearer token required (except public endpoints)

---

## Authentication

All authenticated endpoints require:
```
Authorization: Bearer YOUR_ACCESS_TOKEN
```

---

## Users

### POST /auth/register

Create a new user account.

**Request Body**:
```json
{
  "username": "string (3-20 chars, required)",
  "email": "string (valid email, required)",
  "password": "string (min 8 chars, required)",
  "displayName": "string (optional)"
}
```

**Response** (201):
```json
{
  "id": "uuid",
  "username": "johndoe",
  "email": "john@example.com",
  "displayName": "John Doe",
  "createdAt": "November 5, 2024T10:00:00Z"
}
```

**Errors**:
- `400`: Validation error
- `409`: Email or username already exists

---

### POST /auth/login

Authenticate and receive access token.

**Request Body**:
```json
{
  "email": "string",
  "password": "string"
}
```

**Response** (200):
```json
{
  "accessToken": "jwt-token",
  "refreshToken": "refresh-token",
  "user": {
    "id": "uuid",
    "username": "johndoe",
    "email": "john@example.com"
  }
}
```

---

## Posts

### GET /posts

List all published posts with pagination.

**Query Parameters**:
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `page` | integer | No | 1 | Page number |
| `limit` | integer | No | 20 | Items per page (max 100) |
| `category` | string | No | all | Filter by category |
| `author` | string | No | - | Filter by author username |
| `sort` | string | No | created_desc | Sort: `created_desc`, `created_asc`, `popular` |

**Response** (200):
```json
{
  "data": [
    {
      "id": "uuid",
      "title": "My First Post",
      "slug": "my-first-post",
      "excerpt": "A short excerpt...",
      "coverImage": "https://cdn.blogengine.com/images/abc123.jpg",
      "author": {
        "id": "uuid",
        "username": "johndoe",
        "displayName": "John Doe",
        "avatar": "https://cdn.blogengine.com/avatars/xyz.jpg"
      },
      "category": "Technology",
      "tags": ["nodejs", "api"],
      "publishedAt": "November 5, 2024T10:00:00Z",
      "views": 1250,
      "likes": 42,
      "commentCount": 15
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": false
  }
}
```

---

### GET /posts/:id

Get a single post by ID.

**Parameters**:
- `id`: Post UUID

**Response** (200):
```json
{
  "id": "uuid",
  "title": "My First Post",
  "slug": "my-first-post",
  "content": "Full markdown content...",
  "coverImage": "https://cdn.blogengine.com/images/abc123.jpg",
  "author": {
    "id": "uuid",
    "username": "johndoe",
    "displayName": "John Doe",
    "bio": "Developer and writer",
    "avatar": "https://cdn.blogengine.com/avatars/xyz.jpg"
  },
  "category": "Technology",
  "tags": ["nodejs", "api"],
  "publishedAt": "November 5, 2024T10:00:00Z",
  "updatedAt": "November 5, 2024T12:00:00Z",
  "views": 1250,
  "likes": 42,
  "commentCount": 15,
  "readTime": 8
}
```

**Errors**:
- `404`: Post not found

---

### POST /posts

Create a new post (draft).

**Authentication**: Required  
**Request Body**:
```json
{
  "title": "string (required, max 200)",
  "content": "string (required, markdown)",
  "coverImage": "string (optional, URL)",
  "category": "string (required)",
  "tags": ["string"] (optional, max 10),
  "status": "draft | published (default: draft)"
}
```

**Response** (201):
```json
{
  "id": "uuid",
  "title": "My New Post",
  "slug": "my-new-post",
  "status": "draft",
  "createdAt": "November 5, 2024T10:00:00Z"
}
```

**Errors**:
- `400`: Validation error
- `401`: Not authenticated
- `413`: Content too large

---

### PUT /posts/:id

Update an existing post.

**Authentication**: Required (must be author)

**Request Body**: Same as POST /posts

**Response** (200): Updated post object

**Errors**:
- `400`: Validation error
- `401`: Not authenticated
- `403`: Not authorized (not the author)
- `404`: Post not found

---

### DELETE /posts/:id

Delete a post.

**Authentication**: Required (must be author or admin)

**Response** (204): No content

**Errors**:
- `401`: Not authenticated
- `403`: Not authorized
- `404`: Post not found

---

## Comments

### GET /posts/:postId/comments

Get comments for a post.

**Query Parameters**:
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Page number |
| `limit` | integer | 20 | Items per page |
| `sort` | string | newest | `newest`, `oldest`, `popular` |

**Response** (200):
```json
{
  "data": [
    {
      "id": "uuid",
      "content": "Great post!",
      "author": {
        "id": "uuid",
        "username": "janedoe",
        "displayName": "Jane Doe",
        "avatar": "url"
      },
      "createdAt": "November 5, 2024T11:00:00Z",
      "likes": 5,
      "replyCount": 2
    }
  ],
  "pagination": {...}
}
```

---

### POST /posts/:postId/comments

Add a comment to a post.

**Authentication**: Required

**Request Body**:
```json
{
  "content": "string (required, max 2000)",
  "parentId": "uuid (optional, for replies)"
}
```

**Response** (201): Created comment object

---

## Rate Limiting

- **Limit**: 100 requests per minute per IP
- **Headers**:
  - `X-RateLimit-Limit`: Total allowed
  - `X-RateLimit-Remaining`: Remaining requests
  - `X-RateLimit-Reset`: Reset timestamp

**When exceeded** (429):
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Please try again later.",
    "retryAfter": 60
  }
}
```

---

## Error Format

All errors follow this format:
```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message",
    "details": {}
  }
}
```

**Common Error Codes**:
- `VALIDATION_ERROR`: Invalid input data
- `NOT_FOUND`: Resource doesn't exist
- `UNAUTHORIZED`: Authentication required
- `FORBIDDEN`: Insufficient permissions
- `RATE_LIMIT_EXCEEDED`: Too many requests
- `INTERNAL_ERROR`: Server error

---

**See Also**:
- [Mobile API Reference](mobile-api.md)
- [Admin API Reference](admin-api.md)
- [Authentication Guide](../../explanation/concepts/auth-model.md)

---

**Maintained By**: Backend Team  
**Last Reviewed**: November 5, 2024
