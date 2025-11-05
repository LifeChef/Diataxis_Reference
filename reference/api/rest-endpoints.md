# REST API Endpoints Reference

> **Document Type**: Reference (Information-Oriented)  
> **Category**: API  
> **Version**: 2.0  
> **Last Updated**: 01/20/2025

## Overview

Complete reference for all REST API endpoints in the system.

**Base URL**: `https://api.yourapp.com/v2`

---

## Authentication

All endpoints require Bearer token authentication:

```
Authorization: Bearer YOUR_ACCESS_TOKEN
```

---

## Users

### GET /users/:id

Get user by ID.

**Parameters**:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | User ID |

**Response**:
```json
{
  "id": "user123",
  "name": "John Doe",
  "email": "john@example.com",
  "created_at": "2024-01-01T00:00:00Z"
}
```

**Status Codes**:
- `200`: Success
- `404`: User not found
- `401`: Unauthorized

---

### POST /users

Create new user.

**Request Body**:
```json
{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "password": "secure_password"
}
```

**Response**:
```json
{
  "id": "user124",
  "name": "Jane Doe",
  "email": "jane@example.com"
}
```

**Status Codes**:
- `201`: Created
- `400`: Validation error
- `409`: Email already exists

---

### PUT /users/:id

Update existing user.

**Parameters**:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | User ID |

**Request Body**:
```json
{
  "name": "Updated Name",
  "email": "newemail@example.com"
}
```

---

### DELETE /users/:id

Delete user account.

**Response**: `204 No Content`

---

## Projects

### GET /projects

List all projects.

**Query Parameters**:

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `page` | integer | No | 1 | Page number |
| `limit` | integer | No | 20 | Items per page |
| `status` | string | No | all | Filter by status |

**Response**:
```json
{
  "data": [
    {
      "id": "proj1",
      "name": "Project Alpha",
      "status": "active"
    }
  ],
  "pagination": {
    "page": 1,
    "total_pages": 5,
    "total_items": 100
  }
}
```

---

## Error Responses

All errors follow this format:

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable message",
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

---

## Rate Limiting

- **Limit**: 1000 requests per hour
- **Headers**: 
  - `X-RateLimit-Limit`: Total allowed
  - `X-RateLimit-Remaining`: Remaining requests
  - `X-RateLimit-Reset`: Reset timestamp

---

**See Also**: [Authentication Guide](authentication.md)
