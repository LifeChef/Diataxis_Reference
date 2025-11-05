# Logging Standards

> **Document Type**: Protocol  
> **Audience**: All developers  
> **Last Updated**: November 5, 2024  
> **Status**: ✅ Approved & Enforced

---

## Overview

This document defines standard logging practices for all Blog Engine services to ensure consistent, searchable, and actionable logs.

**All services MUST follow these logging standards.**

---

## Log Levels

| Level | Purpose | When to Use | Example |
|-------|---------|-------------|---------|
| **ERROR** | Critical failures | Service errors, exceptions, failed operations | Database connection failed |
| **WARN** | Potential issues | Deprecated API use, slow queries, retries | Query took 5 seconds |
| **INFO** | Important events | Service start/stop, requests, business events | User registered |
| **DEBUG** | Detailed information | Variable values, flow tracking | Processing item 5 of 100 |
| **TRACE** | Very detailed | Deep debugging, performance profiling | Function entry/exit |

### Usage Guidelines

**Production**: ERROR, WARN, INFO only  
**Staging**: ERROR, WARN, INFO, DEBUG  
**Development**: All levels

---

## Structured Logging Format

### Standard Log Entry

```json
{
  "timestamp": "2024-11-05T14:30:00.123Z",
  "level": "info",
  "message": "User logged in successfully",
  "service": {
    "name": "auth-service",
    "version": "1.5.0",
    "instance": "auth-pod-abc123"
  },
  "request": {
    "id": "req_abc123def456",
    "method": "POST",
    "path": "/api/auth/login",
    "ip": "192.168.1.100"
  },
  "user": {
    "id": 123,
    "email": "user@example.com"
  },
  "duration": 245,
  "statusCode": 200,
  "context": {
    "action": "login",
    "result": "success"
  }
}
```

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `timestamp` | string | ISO 8601 UTC timestamp with milliseconds |
| `level` | string | Log level (error, warn, info, debug, trace) |
| `message` | string | Human-readable log message |
| `service.name` | string | Service name |
| `service.version` | string | Service version |

### Recommended Fields

| Field | Type | Description |
|-------|------|-------------|
| `request.id` | string | Unique request ID for tracing |
| `request.method` | string | HTTP method |
| `request.path` | string | Request path |
| `user.id` | number | Authenticated user ID |
| `duration` | number | Operation duration in milliseconds |
| `error` | object | Error details (for ERROR level) |

---

## Implementation

### Winston (Node.js)

```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: {
    service: {
      name: process.env.SERVICE_NAME,
      version: process.env.SERVICE_VERSION,
      instance: process.env.HOSTNAME
    }
  },
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ 
      filename: 'error.log', 
      level: 'error' 
    }),
    new winston.transports.File({ 
      filename: 'combined.log' 
    })
  ]
});

module.exports = logger;
```

### Request Logging Middleware

```javascript
const logger = require('./logger');

function requestLogger(req, res, next) {
  const start = Date.now();

  // Add request ID
  req.id = req.headers['x-request-id'] || uuidv4();
  
  res.on('finish', () => {
    const duration = Date.now() - start;

    logger.info('Request completed', {
      request: {
        id: req.id,
        method: req.method,
        path: req.path,
        ip: req.ip,
        userAgent: req.get('user-agent')
      },
      user: {
        id: req.user?.id,
        email: req.user?.email
      },
      response: {
        statusCode: res.statusCode,
        duration
      }
    });
  });

  next();
}

module.exports = requestLogger;
```

---

## Log Message Guidelines

### DO ✅

**Good Messages** (specific, actionable):
```javascript
logger.info('Post published successfully', {
  postId: 157,
  authorId: 123,
  publishedAt: new Date().toISOString()
});

logger.error('Database query failed', {
  query: 'SELECT * FROM posts WHERE id = ?',
  error: err.message,
  duration: 5000
});

logger.warn('Slow query detected', {
  query: 'SELECT * FROM posts JOIN users',
  duration: 3500,
  threshold: 1000
});
```

### DON'T ❌

**Bad Messages** (vague, not actionable):
```javascript
logger.info('Success'); // Too vague

logger.error('Error'); // No context

logger.debug('Here'); // Meaningless
```

---

## Error Logging

### Standard Error Format

```javascript
logger.error('Database connection failed', {
  error: {
    message: err.message,
    stack: err.stack,
    code: err.code,
    errno: err.errno
  },
  context: {
    operation: 'database_connect',
    database: 'blogengine',
    host: 'db.example.com'
  },
  request: {
    id: req.id,
    path: req.path
  }
});
```

### Error Levels

- **ERROR**: Errors that affect functionality
- **WARN**: Errors that were handled/recovered
- **INFO**: Expected errors (404, validation failures)

---

## Security & Privacy

### What NOT to Log

❌ **Never log**:
- Passwords (plain or hashed)
- API keys or secrets
- JWT tokens
- Credit card numbers
- Social security numbers
- Full email addresses in production (mask: u***@example.com)

### Sanitization Example

```javascript
function sanitizeUser(user) {
  return {
    id: user.id,
    email: maskEmail(user.email),
    // Don't include: password, token, ssn
  };
}

function maskEmail(email) {
  const [local, domain] = email.split('@');
  return `${local[0]}***@${domain}`;
}

logger.info('User registered', {
  user: sanitizeUser(user)
});
```

---

## Request Tracing

### Request ID Propagation

```javascript
// Generate or extract request ID
const requestId = req.headers['x-request-id'] || uuidv4();

// Add to request object
req.id = requestId;

// Add to response headers
res.setHeader('X-Request-ID', requestId);

// Include in all logs for this request
logger.info('Processing request', {
  request: { id: requestId }
});
```

### Distributed Tracing

```javascript
// Propagate request ID to downstream services
await axios.get('http://post-service/api/posts', {
  headers: {
    'X-Request-ID': req.id,
    'X-Correlation-ID': req.correlationId
  }
});
```

---

## Performance Logging

### Operation Duration

```javascript
const start = Date.now();

const result = await someOperation();

const duration = Date.now() - start;

logger.info('Operation completed', {
  operation: 'fetch_posts',
  duration,
  count: result.length
});

// Alert if slow
if (duration > 1000) {
  logger.warn('Slow operation detected', {
    operation: 'fetch_posts',
    duration,
    threshold: 1000
  });
}
```

---

## Contextual Logging

### Adding Context

```javascript
// Create child logger with context
const childLogger = logger.child({
  user: { id: req.user.id },
  request: { id: req.id }
});

// All logs from child include context
childLogger.info('Fetching user posts'); 
childLogger.info('Posts retrieved', { count: 10 });
```

---

## Log Aggregation

### ELK Stack Integration

Logs automatically sent to Elasticsearch via Filebeat.

**Index Pattern**: `blog-engine-{service}-{date}`

**Example**: `blog-engine-post-service-2024.11.05`

### Kibana Queries

```
# Find all errors for request
request.id: "req_abc123"

# Find slow queries
duration > 1000 AND level: "warn"

# Find user activity
user.id: 123 AND level: "info"
```

---

## Retention Policy

| Environment | Retention | Storage |
|-------------|-----------|---------|
| Production | 90 days | Elasticsearch |
| Staging | 30 days | Elasticsearch |
| Development | 7 days | Local files |

---

## Best Practices

### DO ✅

- Use structured logging (JSON)
- Include request ID in all logs
- Log at appropriate levels
- Include relevant context
- Sanitize sensitive data
- Use consistent field names
- Log operation start AND end
- Include duration metrics

### DON'T ❌

- Log sensitive information
- Use console.log in production
- Log at DEBUG level in production
- Create massive log entries (>10KB)
- Log in tight loops
- Ignore log aggregation
- Use inconsistent formats

---

## Examples by Scenario

### User Registration

```javascript
logger.info('User registration started', {
  request: { id: req.id },
  email: maskEmail(req.body.email)
});

// ... process registration ...

logger.info('User registered successfully', {
  request: { id: req.id },
  user: { id: newUser.id },
  duration: Date.now() - start
});
```

### Database Query

```javascript
const start = Date.now();

logger.debug('Executing database query', {
  query: 'SELECT * FROM posts WHERE authorId = ?',
  params: [authorId]
});

const posts = await db.query(/* ... */);

logger.info('Database query completed', {
  query: 'posts_by_author',
  duration: Date.now() - start,
  rowCount: posts.length
});
```

### API Call to External Service

```javascript
logger.info('Calling external API', {
  service: 'sendgrid',
  endpoint: '/v3/mail/send',
  request: { id: req.id }
});

try {
  const response = await sendgridClient.send(email);
  
  logger.info('External API call successful', {
    service: 'sendgrid',
    statusCode: response.statusCode,
    duration: Date.now() - start
  });
} catch (error) {
  logger.error('External API call failed', {
    service: 'sendgrid',
    error: {
      message: error.message,
      statusCode: error.response?.status
    },
    duration: Date.now() - start
  });
}
```

### Cache Operations

```javascript
const cacheKey = `post:${postId}`;

logger.debug('Cache lookup', { key: cacheKey });

const cached = await redis.get(cacheKey);

if (cached) {
  logger.info('Cache hit', { 
    key: cacheKey,
    ttl: await redis.ttl(cacheKey)
  });
} else {
  logger.info('Cache miss', { key: cacheKey });
  // Fetch from database and cache
}
```

---

## Monitoring & Alerts

### Metrics from Logs

Track these metrics from logs:
- Error rate per service
- Request count per endpoint
- Average response time
- Slow query count
- Cache hit/miss ratio

### Alert Conditions

```yaml
# Alert if error rate > 5%
- alert: HighErrorRate
  expr: |
    rate(logs_total{level="error"}[5m]) 
    / rate(logs_total[5m]) > 0.05

# Alert if slow queries increase
- alert: SlowQueries
  expr: |
    rate(logs_total{level="warn",message=~".*slow.*"}[5m]) > 10
```

---

## Related Documentation

- [Error Handling Standards](error-handling.md)
- [Event Schema](event-schema.md)
- [API Conventions](api-conventions.md)

---

**Maintained By**: Platform Team  
**Review Frequency**: Quarterly  
**Next Review**: February 1, 2025
