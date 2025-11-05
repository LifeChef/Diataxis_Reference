# Data Protection

> **Document Type**: Security Control  
> **SOC2 Controls**: CC6.1, CC6.6, CC6.7  
> **Last Updated**: November 5, 2024  
> **Status**: ✅ Implemented

---

## Overview

This document outlines data protection measures for the Blog Engine system to ensure confidentiality, integrity, and availability of sensitive data.

---

## Encryption at Rest

### Database Encryption

**PostgreSQL TDE (Transparent Data Encryption)**
- All databases encrypted using AES-256
- Encryption keys managed by AWS KMS
- Automatic key rotation every 90 days

```sql
-- Verify encryption status
SELECT datname, pg_database.oid 
FROM pg_database 
WHERE datistemplate = false;

-- All user databases encrypted by default
```

### File Storage Encryption

**S3 Server-Side Encryption (SSE-KMS)**
```javascript
await s3.putObject({
  Bucket: 'blog-engine-uploads',
  Key: key,
  Body: fileBuffer,
  ServerSideEncryption: 'aws:kms',
  SSEKMSKeyId: process.env.KMS_KEY_ID
});
```

### Backup Encryption

- All database backups encrypted with AES-256
- Stored in encrypted S3 buckets
- 30-day retention, encrypted snapshots

---

## Encryption in Transit

### TLS/SSL Requirements

**Minimum Version**: TLS 1.2  
**Preferred Version**: TLS 1.3

```nginx
# NGINX SSL configuration
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers HIGH:!aNULL:!MD5;
ssl_prefer_server_ciphers on;

# HSTS
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```

### Certificate Management

- **Provider**: Let's Encrypt / AWS Certificate Manager
- **Auto-renewal**: 30 days before expiration
- **Monitoring**: Certificate expiry alerts

### Internal Service Communication

```yaml
# Kubernetes NetworkPolicy - enforce TLS
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: require-tls
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector: {}
    ports:
    - protocol: TCP
      port: 443
```

---

## Secrets Management

### AWS Secrets Manager

**Storage**: All secrets in AWS Secrets Manager
**Access**: IAM role-based, least privilege
**Rotation**: Automatic every 90 days

```javascript
const AWS = require('aws-sdk');
const secretsManager = new AWS.SecretsManager();

async function getSecret(secretName) {
  const data = await secretsManager.getSecretValue({
    SecretId: secretName
  }).promise();
  
  return JSON.parse(data.SecretString);
}

// Usage
const dbCreds = await getSecret('prod/database/credentials');
```

### Environment Variables

**DO NOT** store secrets in:
- ❌ `.env` files committed to Git
- ❌ Docker images
- ❌ ConfigMaps (use Secrets instead)

**DO** use:
- ✅ AWS Secrets Manager
- ✅ Kubernetes Secrets (encrypted at rest)
- ✅ Vault (if using HashiCorp Vault)

```yaml
# Kubernetes Secret from AWS Secrets Manager
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: <base64-encoded>
  password: <base64-encoded>
```

---

## Data Classification

### Classification Levels

| Level | Examples | Encryption | Access |
|-------|----------|------------|--------|
| **Public** | Blog posts, public profiles | Optional | Anyone |
| **Internal** | Analytics, logs | At rest | Employees |
| **Confidential** | User emails, IP addresses | At rest + transit | Need-to-know |
| **Restricted** | Passwords, tokens, payment info | At rest + transit + hashed | Minimal access |

### Handling by Classification

```javascript
// Restricted data - never log or expose
const password = req.body.password; // Hash immediately
const hashedPassword = await bcrypt.hash(password, 12);

// Confidential data - mask in logs
logger.info('User registered', {
  email: maskEmail(user.email), // u***@example.com
  userId: user.id
});

// Internal data - log with caution
logger.info('Query executed', {
  duration: 145,
  rowCount: 10
  // Don't log: query params with PII
});
```

---

## Data Sanitization

### Input Sanitization

```javascript
const DOMPurify = require('isomorphic-dompurify');
const validator = require('validator');

// Sanitize HTML content
const cleanContent = DOMPurify.sanitize(userInput, {
  ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'a'],
  ALLOWED_ATTR: ['href']
});

// Validate and sanitize email
if (!validator.isEmail(email)) {
  throw new ValidationError('Invalid email');
}
const normalizedEmail = validator.normalizeEmail(email);

// Escape SQL (use parameterized queries)
const posts = await db.query(
  'SELECT * FROM posts WHERE author_id = ?',
  [authorId] // Parameterized - prevents SQL injection
);
```

### Output Encoding

```javascript
// HTML entity encoding
const escape = require('escape-html');
const safeTitle = escape(post.title);

// JSON response (automatically encoded by Express)
res.json({ title: post.title }); // Safe
```

---

## Data Retention

### Retention Policies

| Data Type | Retention | Deletion Method |
|-----------|-----------|-----------------|
| User accounts | Until deletion request | Hard delete after 30-day grace |
| Blog posts | Indefinite (or until user deletes) | Soft delete, hard after 90 days |
| Access logs | 90 days | Auto-delete by date |
| Audit logs | 7 years (compliance) | Archive to S3 Glacier |
| Session data | 24 hours | Auto-expire in Redis |
| Deleted user data | 30 days (backup retention) | Permanent deletion |

### Automated Deletion

```javascript
// Cron job to delete old soft-deleted posts
cron.schedule('0 2 * * *', async () => {
  const ninetyDaysAgo = new Date();
  ninetyDaysAgo.setDate(ninetyDaysAgo.getDate() - 90);
  
  const result = await db('posts')
    .where('deleted_at', '<', ninetyDaysAgo)
    .where('status', 'deleted')
    .delete();
  
  logger.info('Deleted old posts', { count: result });
});
```

---

## Personal Data (GDPR/Privacy)

### Data Subject Rights

**Right to Access**: Users can download their data
```javascript
// GET /api/users/me/data-export
async function exportUserData(userId) {
  const data = {
    profile: await getUserProfile(userId),
    posts: await getUserPosts(userId),
    comments: await getUserComments(userId),
    settings: await getUserSettings(userId)
  };
  
  // Generate JSON export
  return JSON.stringify(data, null, 2);
}
```

**Right to Erasure**: Users can request deletion
```javascript
// DELETE /api/users/me
async function deleteUser(userId) {
  await db.transaction(async (trx) => {
    // Anonymize posts (keep content, remove attribution)
    await trx('posts')
      .where('author_id', userId)
      .update({ author_id: null, author_name: 'Deleted User' });
    
    // Delete comments
    await trx('comments').where('user_id', userId).delete();
    
    // Delete user record
    await trx('users').where('id', userId).delete();
  });
  
  logger.info('User deleted', { userId });
}
```

**Right to Portability**: Provide data in machine-readable format (JSON)

---

## Password Security

### Password Requirements

- Minimum 8 characters
- Must include: uppercase, lowercase, number, special character
- No common passwords (check against breach database)
- Password history: prevent reuse of last 5 passwords

### Password Hashing

```javascript
const bcrypt = require('bcrypt');

// Hash password (cost factor: 12)
const hash = await bcrypt.hash(password, 12);

// Verify password
const isValid = await bcrypt.compare(password, hash);

// Store in database
await db('users').insert({
  email,
  password_hash: hash // Never store plain passwords
});
```

### Password Reset

```javascript
const crypto = require('crypto');

// Generate secure reset token
const resetToken = crypto.randomBytes(32).toString('hex');
const resetTokenHash = crypto
  .createHash('sha256')
  .update(resetToken)
  .digest('hex');

// Store hashed token with expiry
await db('password_resets').insert({
  user_id: userId,
  token_hash: resetTokenHash,
  expires_at: new Date(Date.now() + 3600000) // 1 hour
});

// Email unhashed token to user
await emailService.sendPasswordReset(email, resetToken);
```

---

## Token Security

### JWT Best Practices

```javascript
const jwt = require('jsonwebtoken');

// Generate access token (short-lived)
const accessToken = jwt.sign(
  { userId: user.id, role: user.role },
  process.env.JWT_SECRET,
  { expiresIn: '15m' }
);

// Generate refresh token (long-lived)
const refreshToken = jwt.sign(
  { userId: user.id },
  process.env.REFRESH_TOKEN_SECRET,
  { expiresIn: '7d' }
);

// Store refresh token hash
const refreshTokenHash = crypto
  .createHash('sha256')
  .update(refreshToken)
  .digest('hex');

await db('refresh_tokens').insert({
  user_id: user.id,
  token_hash: refreshTokenHash,
  expires_at: new Date(Date.now() + 7 * 24 * 3600000)
});
```

### Token Rotation

- Access tokens: 15 minutes
- Refresh tokens: 7 days, rotated on use
- Revoke all tokens on password change

---

## Data Masking

### PII Masking in Logs

```javascript
function maskEmail(email) {
  const [local, domain] = email.split('@');
  return `${local[0]}***@${domain}`;
}

function maskPhone(phone) {
  return phone.replace(/\d(?=\d{4})/g, '*');
}

function maskIP(ip) {
  const parts = ip.split('.');
  return `${parts[0]}.${parts[1]}.***.*
**`;
}

// Usage
logger.info('User action', {
  email: maskEmail(user.email),
  ip: maskIP(req.ip)
});
```

### Database Masking (Non-Production)

```sql
-- Anonymize data in staging/dev
UPDATE users 
SET 
  email = CONCAT('user', id, '@example.com'),
  name = CONCAT('User ', id),
  phone = NULL
WHERE environment = 'staging';
```

---

## Backup & Recovery

### Backup Strategy

**Frequency**: 
- Database: Daily automated snapshots
- Files: Continuous sync to S3 with versioning

**Retention**:
- Daily: 7 days
- Weekly: 4 weeks
- Monthly: 12 months

**Encryption**: All backups encrypted with AES-256

```bash
# PostgreSQL backup with encryption
pg_dump -h $DB_HOST -U $DB_USER $DB_NAME | \
  gpg --encrypt --recipient backup@blogengine.com | \
  aws s3 cp - s3://blog-engine-backups/daily/$(date +%F).sql.gpg
```

### Disaster Recovery

**RTO** (Recovery Time Objective): 4 hours  
**RPO** (Recovery Point Objective): 1 hour

**Recovery Procedure**:
1. Restore from latest snapshot
2. Apply transaction logs
3. Verify data integrity
4. Switch DNS to backup region

---

## Monitoring & Auditing

### Security Events to Log

- All authentication attempts (success/failure)
- Authorization failures
- Data access (for restricted data)
- Configuration changes
- Admin actions
- Failed API requests (potential attacks)

```javascript
// Audit log example
logger.info('Sensitive data accessed', {
  action: 'user.data.export',
  userId: req.user.id,
  targetUserId: targetId,
  requestId: req.id,
  ip: req.ip,
  timestamp: new Date().toISOString()
});
```

### Anomaly Detection

Monitor for:
- Unusual data access patterns
- Failed login attempts (brute force)
- Large data exports
- Off-hours admin actions

---

## Compliance

### SOC2 Type II Controls

- **CC6.1**: Logical and physical access controls
- **CC6.6**: Encryption of data at rest and in transit
- **CC6.7**: Deletion of data per retention policies

### GDPR Requirements

- ✅ Data encryption
- ✅ Right to access (data export)
- ✅ Right to erasure (account deletion)
- ✅ Data portability (JSON format)
- ✅ Breach notification (within 72 hours)

---

## Related Documentation

- [Authentication Security](authentication.md)
- [Authorization](authorization.md)
- [API Security](api-security.md)

---

**Security Contact**: security@blogengine.com  
**Incident Response**: Available 24/7  
**Last Security Audit**: November 1, 2024
