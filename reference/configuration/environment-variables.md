# Environment Variables Reference

> **Document Type**: Reference (Information-Oriented)  
> **Category**: Configuration  
> **Version**: 1.0  
> **Last Updated**: 01/20/2025

## Overview

Complete list of environment variables for application configuration.

---

## Required Variables

### NODE_ENV

**Type**: `string`  
**Required**: Yes  
**Valid Values**: `development`, `staging`, `production`  
**Description**: Application environment mode

**Example**:
```
NODE_ENV=production
```

---

### DATABASE_URL

**Type**: `string` (URI format)  
**Required**: Yes  
**Description**: PostgreSQL database connection string

**Format**:
```
postgresql://[user]:[password]@[host]:[port]/[database]
```

**Example**:
```
DATABASE_URL=postgresql://admin:secret@localhost:5432/myapp
```

---

### JWT_SECRET

**Type**: `string`  
**Required**: Yes  
**Minimum Length**: 32 characters  
**Description**: Secret key for JWT token signing

**Example**:
```
JWT_SECRET=your-256-bit-secret-key-here
```

---

## Optional Variables

### PORT

**Type**: `integer`  
**Required**: No  
**Default**: `3000`  
**Range**: `1024-65535`  
**Description**: Port number for server

---

### LOG_LEVEL

**Type**: `string`  
**Required**: No  
**Default**: `info`  
**Valid Values**: `error`, `warn`, `info`, `debug`  
**Description**: Logging verbosity level

---

### REDIS_URL

**Type**: `string` (URI format)  
**Required**: No  
**Default**: `redis://localhost:6379`  
**Description**: Redis cache connection

---

### MAX_UPLOAD_SIZE

**Type**: `integer` (bytes)  
**Required**: No  
**Default**: `10485760` (10MB)  
**Description**: Maximum file upload size

---

## Feature Flags

### FEATURE_NEW_UI

**Type**: `boolean`  
**Default**: `false`  
**Description**: Enable new UI components

---

### ENABLE_ANALYTICS

**Type**: `boolean`  
**Default**: `true`  
**Description**: Enable analytics tracking

---

## Example .env File

```bash
# Application
NODE_ENV=development
PORT=3000

# Database
DATABASE_URL=postgresql://localhost:5432/devdb

# Security
JWT_SECRET=your-secret-key-min-32-chars
CORS_ORIGIN=http://localhost:3001

# External Services
REDIS_URL=redis://localhost:6379
SENDGRID_API_KEY=SG.xxx

# Features
FEATURE_NEW_UI=true
ENABLE_ANALYTICS=false
```

---

## Security Notes

- Never commit `.env` files to version control
- Use different secrets for each environment
- Rotate secrets regularly
- Use secret management tools in production

---

**See Also**: [Deployment Guide](../../how-to/deployment/deploy-to-production.md)
