# API Security

> **Document Type**: Security Control  
> **SOC2 Controls**: CC6.1, CC6.6, CC7.2  
> **Last Updated**: November 5, 2024  
> **Status**: ✅ Implemented

---

## Overview

This document outlines API security controls for the Blog Engine to protect against common web vulnerabilities and attacks.

---

## Rate Limiting

### Rate Limit Configuration

| User Type | Limit | Window | Burst |
|-----------|-------|--------|-------|
| Anonymous | 100 req/hour | 1 hour | 10 req/min |
| Authenticated | 1,000 req/hour | 1 hour | 50 req/min |
| Admin | 5,000 req/hour | 1 hour | 100 req/min |
| API Keys | Custom per key | 1 hour | Custom |

### Implementation

```javascript
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');

const limiter = rateLimit({
  store: new RedisStore({
    client: redisClient
  }),
  windowMs: 60 * 60 * 1000, // 1 hour
  max: async (req) => {
    if (!req.user) return 100; // Anonymous
    if (req.user.role === 'admin') return 5000;
    return 1000; // Authenticated
  },
  standardHeaders: true,
  legacyHeaders: false,
  handler: (req, res) => {
    res.status(429).json({
      error: {
        code: 'RATE_LIMIT_EXCEEDED',
        message: 'Too many requests. Please try again later.',
        retryAfter: res.getHeader('Retry-After')
      }
    });
  }
});

app.use('/api/', limiter);
```

### Response Headers

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1699203600
Retry-After: 3600
```

---

## Input Validation

### Request Validation Middleware

```javascript
const { body, param, query, validationResult } = require('express-validator');

// Validation rules for creating a post
const validateCreatePost = [
  body('title')
    .trim()
    .isLength({ min: 1, max: 200 })
    .withMessage('Title must be 1-200 characters'),
  
  body('content')
    .trim()
    .isLength({ min: 100 })
    .withMessage('Content must be at least 100 characters'),
  
  body('tags')
    .optional()
    .isArray({ max: 5 })
    .withMessage('Maximum 5 tags allowed'),
  
  body('tags.*')
    .trim()
    .isLength({ min: 1, max: 50 })
    .matches(/^[a-z0-9-]+$/)
    .withMessage('Tags must be lowercase alphanumeric with hyphens'),
  
  body('status')
    .optional()
    .isIn(['draft', 'published'])
    .withMessage('Status must be draft or published'),
  
  // Check validation results
  (req, res, next) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(422).json({
        error: {
          code: 'VALIDATION_ERROR',
          message: 'Request validation failed',
          details: errors.array().map(err => ({
            field: err.param,
            message: err.msg,
            rejectedValue: err.value
          }))
        }
      });
    }
    next();
  }
];

// Use in route
app.post('/api/posts', 
  authenticate,
  validateCreatePost,
  async (req, res) => {
    // Request is validated
    const post = await postService.create(req.user.id, req.body);
    res.status(201).json(post);
  }
);
```

### SQL Injection Prevention

```javascript
// BAD: String concatenation
const query = `SELECT * FROM posts WHERE author_id = ${authorId}`; // ❌ SQL Injection!

// GOOD: Parameterized queries
const posts = await db.query(
  'SELECT * FROM posts WHERE author_id = ?',
  [authorId] // ✅ Safe
);

// GOOD: Query builder (Knex)
const posts = await db('posts')
  .where('author_id', authorId); // ✅ Automatically parameterized
```

### XSS Prevention

```javascript
const DOMPurify = require('isomorphic-dompurify');

// Sanitize HTML content
function sanitizeHTML(html) {
  return DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'u', 'a', 'ul', 'ol', 'li'],
    ALLOWED_ATTR: ['href', 'title'],
    ALLOW_DATA_ATTR: false
  });
}

// Use in post creation
const cleanContent = sanitizeHTML(req.body.content);
```

---

## CORS (Cross-Origin Resource Sharing)

### Configuration

```javascript
const cors = require('cors');

const corsOptions = {
  origin: (origin, callback) => {
    const allowedOrigins = [
      'https://blogengine.com',
      'https://www.blogengine.com',
      'https://admin.blogengine.com'
    ];
    
    // Allow localhost in development
    if (process.env.NODE_ENV === 'development') {
      allowedOrigins.push(/^http:\/\/localhost:\d+$/);
    }
    
    if (!origin || allowedOrigins.some(allowed => 
      typeof allowed === 'string' ? allowed === origin : allowed.test(origin)
    )) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-Request-ID'],
  exposedHeaders: ['X-Request-ID', 'X-RateLimit-Remaining'],
  maxAge: 86400 // 24 hours
};

app.use(cors(corsOptions));
```

---

## Security Headers

### Helmet.js Implementation

```javascript
const helmet = require('helmet');

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https://cdn.blogengine.com"],
      connectSrc: ["'self'", "https://api.blogengine.com"],
      fontSrc: ["'self'"],
      objectSrc: ["'none'"],
      upgradeInsecureRequests: []
    }
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  },
  frameguard: {
    action: 'deny'
  },
  noSniff: true,
  xssFilter: true
}));
```

### Response Headers

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'; ...
```

---

## Authentication Security

### JWT Token Validation

```javascript
const jwt = require('jsonwebtoken');

async function authenticateJWT(req, res, next) {
  const authHeader = req.headers.authorization;
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({
      error: {
        code: 'MISSING_AUTHORIZATION',
        message: 'Authorization header required'
      }
    });
  }
  
  const token = authHeader.substring(7);
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    
    // Check if token is revoked
    const revoked = await isTokenRevoked(decoded.jti);
    if (revoked) {
      return res.status(401).json({
        error: {
          code: 'TOKEN_REVOKED',
          message: 'Token has been revoked'
        }
      });
    }
    
    // Load user
    req.user = await db('users').where('id', decoded.userId).first();
    
    if (!req.user) {
      return res.status(401).json({
        error: {
          code: 'USER_NOT_FOUND',
          message: 'User no longer exists'
        }
      });
    }
    
    next();
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      return res.status(401).json({
        error: {
          code: 'TOKEN_EXPIRED',
          message: 'Token has expired'
        }
      });
    }
    
    return res.status(401).json({
      error: {
        code: 'INVALID_TOKEN',
        message: 'Invalid authentication token'
      }
    });
  }
}
```

---

## Request ID Tracking

### Implementation

```javascript
const { v4: uuidv4 } = require('uuid');

function requestIdMiddleware(req, res, next) {
  // Use existing request ID or generate new one
  req.id = req.headers['x-request-id'] || uuidv4();
  
  // Add to response headers
  res.setHeader('X-Request-ID', req.id);
  
  // Add to all logs
  req.log = logger.child({ requestId: req.id });
  
  next();
}

app.use(requestIdMiddleware);
```

---

## File Upload Security

### Configuration

```javascript
const multer = require('multer');
const path = require('path');

const upload = multer({
  limits: {
    fileSize: 5 * 1024 * 1024, // 5MB
    files: 1
  },
  fileFilter: (req, file, cb) => {
    // Allow only specific file types
    const allowedTypes = /jpeg|jpg|png|gif|webp/;
    const extname = allowedTypes.test(
      path.extname(file.originalname).toLowerCase()
    );
    const mimetype = allowedTypes.test(file.mimetype);
    
    if (extname && mimetype) {
      return cb(null, true);
    }
    
    cb(new Error('Invalid file type. Only images allowed.'));
  }
});

// Use in route
app.post('/api/posts/:id/image',
  authenticate,
  upload.single('image'),
  async (req, res) => {
    // Additional validation
    if (!req.file) {
      return res.status(400).json({
        error: { message: 'No file uploaded' }
      });
    }
    
    // Scan for malware (if applicable)
    // await scanFile(req.file.buffer);
    
    // Process upload
    const url = await uploadToS3(req.file);
    res.json({ url });
  }
);
```

---

## API Versioning

### URL-Based Versioning

```javascript
// v1 routes
app.use('/api/v1', v1Routes);

// v2 routes (with breaking changes)
app.use('/api/v2', v2Routes);

// Default to latest version
app.use('/api', v2Routes);
```

---

## Error Handling

### Secure Error Responses

```javascript
// Error handler middleware
app.use((err, req, res, next) => {
  // Log error with full details
  logger.error('Request error', {
    error: err.message,
    stack: err.stack,
    requestId: req.id
  });
  
  // Send safe error to client
  const statusCode = err.statusCode || 500;
  const errorResponse = {
    error: {
      code: err.code || 'INTERNAL_SERVER_ERROR',
      message: err.message || 'An error occurred',
      requestId: req.id
    }
  };
  
  // Never expose stack traces in production
  if (process.env.NODE_ENV !== 'production' && err.stack) {
    errorResponse.error.stack = err.stack;
  }
  
  res.status(statusCode).json(errorResponse);
});
```

---

## Brute Force Protection

### Login Attempt Limiting

```javascript
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts
  skipSuccessfulRequests: true,
  handler: (req, res) => {
    res.status(429).json({
      error: {
        code: 'TOO_MANY_LOGIN_ATTEMPTS',
        message: 'Too many login attempts. Please try again in 15 minutes.'
      }
    });
  }
});

app.post('/api/auth/login', loginLimiter, async (req, res) => {
  // Login logic
});
```

---

## API Key Management

### Generation & Storage

```javascript
const crypto = require('crypto');

// Generate API key
function generateApiKey() {
  return crypto.randomBytes(32).toString('hex');
}

// Hash for storage
function hashApiKey(apiKey) {
  return crypto
    .createHash('sha256')
    .update(apiKey)
    .digest('hex');
}

// Create API key
async function createApiKey(userId, name, permissions) {
  const apiKey = generateApiKey();
  const keyHash = hashApiKey(apiKey);
  
  await db('api_keys').insert({
    user_id: userId,
    name,
    key_hash: keyHash,
    permissions: JSON.stringify(permissions),
    created_at: new Date(),
    expires_at: new Date(Date.now() + 365 * 24 * 3600000) // 1 year
  });
  
  // Return unhashed key only once
  return apiKey;
}
```

---

## Monitoring & Alerting

### Security Events to Monitor

- Failed authentication attempts
- Rate limit violations
- Invalid input attempts (SQLi, XSS)
- Unauthorized access attempts
- Unusual API usage patterns
- Large file uploads
- Brute force attempts

### Alert Configuration

```javascript
// Alert on high error rate
if (errorRate > 0.05) { // 5%
  alert('High API error rate detected', {
    errorRate,
    endpoint: req.path,
    timestamp: new Date()
  });
}

// Alert on suspicious activity
if (failedLoginAttempts > 10) {
  alert('Potential brute force attack', {
    ip: req.ip,
    userId: req.body.email,
    attempts: failedLoginAttempts
  });
}
```

---

## Security Testing

### Automated Tests

```javascript
describe('API Security', () => {
  it('should reject requests without authentication', async () => {
    await request(app)
      .post('/api/posts')
      .send({ title: 'Test' })
      .expect(401);
  });
  
  it('should prevent SQL injection', async () => {
    const maliciousInput = "1' OR '1'='1";
    
    await request(app)
      .get(`/api/users/${maliciousInput}`)
      .expect(400); // Should fail validation
  });
  
  it('should sanitize XSS attempts', async () => {
    const xssPayload = '<script>alert("XSS")</script>';
    
    const response = await request(app)
      .post('/api/posts')
      .set('Authorization', `Bearer ${token}`)
      .send({
        title: 'Test',
        content: xssPayload
      })
      .expect(201);
    
    expect(response.body.content).not.toContain('<script>');
  });
  
  it('should enforce rate limits', async () => {
    // Make 101 requests (limit is 100)
    for (let i = 0; i < 101; i++) {
      const status = await request(app)
        .get('/api/posts')
        .then(res => res.status);
      
      if (i === 100) {
        expect(status).toBe(429); // Rate limit exceeded
      }
    }
  });
});
```

---

## Best Practices

### DO ✅

- Validate all input server-side
- Use parameterized queries
- Implement rate limiting
- Set secure headers
- Log security events
- Keep dependencies updated
- Use HTTPS everywhere
- Implement CORS properly

### DON'T ❌

- Trust client input
- Expose sensitive errors
- Use weak authentication
- Skip authorization checks
- Ignore security updates
- Store secrets in code
- Allow unlimited requests

---

## Related Documentation

- [Authentication Security](authentication.md)
- [Authorization](authorization.md)
- [Data Protection](data-protection.md)
- [Error Handling Standards](../protocols/error-handling.md)

---

**Security Contact**: security@blogengine.com  
**Vulnerability Reports**: security@blogengine.com  
**Last Security Audit**: November 1, 2024
