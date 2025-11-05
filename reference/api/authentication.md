# Authentication Reference

> **Document Type**: Reference (Information-Oriented)  
> **Category**: API  
> **Version**: 2.0  
> **Last Updated**: 01/20/2025

## Overview

Authentication mechanisms for API access.

**Supported Methods**:
- OAuth 2.0
- API Keys
- JWT Tokens

---

## OAuth 2.0

### Authorization Code Flow

**Step 1: Authorization Request**

```
GET /oauth/authorize?
  response_type=code&
  client_id=YOUR_CLIENT_ID&
  redirect_uri=YOUR_REDIRECT_URI&
  scope=read write
```

**Step 2: Token Exchange**

```
POST /oauth/token
Content-Type: application/json

{
  "grant_type": "authorization_code",
  "code": "AUTH_CODE",
  "client_id": "YOUR_CLIENT_ID",
  "client_secret": "YOUR_CLIENT_SECRET"
}
```

**Response**:
```json
{
  "access_token": "eyJhbGc...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "refresh..."
}
```

---

## API Keys

Create API key:

```
POST /api-keys
Authorization: Bearer YOUR_TOKEN

{
  "name": "My API Key",
  "scopes": ["read", "write"]
}
```

Use in requests:

```
Authorization: ApiKey YOUR_API_KEY
```

---

## JWT Tokens

**Structure**:
```
header.payload.signature
```

**Claims**:
- `sub`: User ID
- `exp`: Expiration timestamp
- `iat`: Issued at
- `scopes`: Permissions array

---

## Scopes

| Scope | Description |
|-------|-------------|
| `read` | Read access |
| `write` | Create/update access |
| `delete` | Delete access |
| `admin` | Full administrative access |

---

**See Also**: [REST Endpoints](rest-endpoints.md)
