# Authentication Security

> **Document Type**: Security Control  
> **SOC2 Controls**: CC6.1, CC6.2  
> **Last Updated**: November 5, 2024  
> **Owner**: Security Team

---

## Overview

This document defines authentication mechanisms, security controls, and implementation requirements for Blog Engine user authentication.

**Supported authentication methods**:
1. Email/password (with bcrypt hashing)
2. OAuth2 (Google, GitHub)
3. Multi-factor authentication (MFA) - for admin accounts

---

## Authentication Flow

### Email/Password Login

```
1. User submits email + password
   ↓
2. API Gateway forwards to Auth Service
   ↓
3. Auth Service:
   - Looks up user by email
   - Verifies password (bcrypt.compare)
   - Checks account status (active/locked)
   - Logs login attempt
   ↓
4. On success:
   - Generate JWT access token (1h TTL)
   - Generate refresh token (7d TTL)
   - Store refresh token in Redis
   - Return tokens to client
   ↓
5. Client stores:
   - Access token: Memory only
   - Refresh token: HttpOnly cookie
```

**Implementation**: Auth Service (port 3001)  
**Endpoints**: `POST /auth/login`  
**Related ADR**: [ADR-003: JWT Authentication](../decisions/adr-003-jwt-auth.md)

---

## Password Security

### Password Requirements

**Minimum requirements** (enforced at registration):
- Length: 12 characters minimum
- Must contain: 1 uppercase, 1 lowercase, 1 number, 1 special char
- Cannot be: Common passwords (checked against breached password list)
- Cannot be: Email address or username

**Implementation**:
```javascript
// Validation regex
const passwordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{12,}$/;

// Check against haveibeenpwned API (k-anonymity)
const isBreached = await checkBreachedPassword(password);
```

### Password Hashing

**Algorithm**: bcrypt  
**Cost factor**: 12 (adjustable for security/performance balance)  
**Salt**: Automatically generated per password (bcrypt handles this)

**Implementation**:
```javascript
const bcrypt = require('bcrypt');
const SALT_ROUNDS = 12;

// Hashing (registration)
const hashedPassword = await bcrypt.hash(password, SALT_ROUNDS);

// Verification (login)
const isValid = await bcrypt.compare(password, hashedPassword);
```

**Never**:
- Log passwords (even hashed)
- Store passwords in plain text
- Send passwords in URLs
- Include passwords in error messages

---

## JWT Token Security

### Access Token

**Purpose**: Authenticate API requests  
**TTL**: 1 hour  
**Storage**: Client memory (never localStorage)  
**Algorithm**: RS256 (RSA public/private key)

**Payload structure**:
```json
{
  "sub": "user-uuid",
  "email": "user@example.com",
  "role": "user|admin",
  "iat": 1699200000,
  "exp": 1699203600,
  "iss": "blog-engine",
  "aud": "blog-engine-api"
}
```

**Security controls**:
- Short TTL (1h) limits exposure window
- RS256 signing (can't be forged without private key)
- Validate `exp`, `iss`, `aud` on every request
- Include in `Authorization: Bearer <token>` header only

### Refresh Token

**Purpose**: Obtain new access tokens  
**TTL**: 7 days  
**Storage**: 
  - Server: Redis (revocable)
  - Client: HttpOnly, Secure, SameSite=Strict cookie

**Security controls**:
- Stored in Redis (can revoke immediately)
- Single-use recommended (rotate on refresh)
- HttpOnly cookie (prevents XSS theft)
- Secure flag (HTTPS only)
- SameSite=Strict (prevents CSRF)

**Redis key format**:
```
refresh:{userId}:{tokenId}
TTL: 7 days
Value: {createdAt, userAgent, ipAddress}
```

### Token Revocation

**Scenarios requiring revocation**:
1. User logs out
2. Password changed
3. Account compromised
4. Role/permissions changed
5. Account deletion

**Implementation**:
```javascript
// Revoke all refresh tokens for user
await redis.del(`refresh:${userId}:*`);

// Emergency: Blocklist access token (rarely used)
await redis.setex(`blocklist:${tokenId}`, 3600, '1'); // TTL = token TTL
```

**Access token blocklist** (emergency only):
- Check Redis on critical operations (password change, delete account)
- Small performance impact, use sparingly
- Automatic cleanup after token expires

---

## OAuth2 Integration

### Supported Providers

| Provider | Status | Use Case | Client ID |
|----------|--------|----------|-----------|
| Google | ✅ Active | Primary OAuth | `*.apps.googleusercontent.com` |
| GitHub | ✅ Active | Developer preference | `Iv1.abc...` |
| Facebook | 📋 Planned | Social login | - |

### OAuth2 Flow (Google Example)

```
1. User clicks "Sign in with Google"
   ↓
2. Redirect to Google OAuth consent screen
   URL: https://accounts.google.com/o/oauth2/v2/auth
   Params: client_id, redirect_uri, scope, state
   ↓
3. User approves, Google redirects back
   Callback: https://api.blog.com/auth/oauth/google/callback?code=...&state=...
   ↓
4. Auth Service:
   - Validates `state` (CSRF protection)
   - Exchanges code for Google access token
   - Fetches user profile from Google
   - Creates or updates user in database
   - Generates Blog Engine JWT tokens
   ↓
5. Return JWT tokens to client (same as email/password login)
```

**Security controls**:
- `state` parameter (random, session-specific, validated on callback)
- Validate OAuth token with provider
- Store minimal OAuth data (provider ID, email)
- User can disconnect OAuth in account settings

**Implementation**: Auth Service `/auth/oauth/:provider` endpoints

---

## Multi-Factor Authentication (MFA)

**Status**: ✅ Implemented (admin accounts), 📋 Planned (optional for all users)

### TOTP (Time-based One-Time Password)

**Method**: Authenticator app (Google Authenticator, Authy)  
**Algorithm**: TOTP (RFC 6238)  
**Required for**: Admin accounts, optional for regular users

**Setup flow**:
```
1. User enables MFA in account settings
   ↓
2. Generate secret key (base32 encoded)
   ↓
3. Display QR code (otpauth:// URI)
   ↓
4. User scans QR code in authenticator app
   ↓
5. User enters 6-digit code to verify setup
   ↓
6. Store encrypted secret in database
   Generate backup codes (10 single-use codes)
   Mark MFA as enabled
```

**Login flow with MFA**:
```
1. User enters email + password (valid)
   ↓
2. Check if MFA enabled
   ↓
3. If yes:
   - Return 200 with `mfa_required: true`
   - Provide temporary token (5 min TTL)
   ↓
4. Client prompts for MFA code
   ↓
5. User submits 6-digit code + temporary token
   ↓
6. Verify TOTP code (allow ±1 time window for clock skew)
   ↓
7. If valid: Return JWT tokens
   If invalid: Increment failed attempt counter
```

**Backup codes**:
- Generate 10 codes at MFA setup
- Single-use, randomly generated (16 characters)
- Display once, user must save securely
- Can regenerate anytime (invalidates old codes)

**Security controls**:
- Rate limit MFA verification (10 attempts/hour)
- Lock account after 10 failed MFA attempts
- Email notification on successful MFA login
- Backup codes stored hashed (bcrypt)

---

## Account Security

### Account Lockout

**Trigger**: 5 failed login attempts within 15 minutes  
**Duration**: 15 minutes (or until password reset)  
**Notification**: Email sent to account holder

**Implementation**:
```javascript
// Redis tracking
Key: `login_attempts:${email}`
Value: attempt count
TTL: 15 minutes

// On failed login
const attempts = await redis.incr(`login_attempts:${email}`);
if (attempts === 1) {
  await redis.expire(`login_attempts:${email}`, 900); // 15 min
}
if (attempts >= 5) {
  // Lock account
  await lockAccount(userId, 'Too many failed attempts');
  await sendEmail(email, 'Account locked');
}
```

### Session Management

**Session defined as**: Active refresh token in Redis

**Limits**:
- Max concurrent sessions per user: 5
- Exceeding limit: Remove oldest session

**Session data tracked**:
```json
{
  "tokenId": "uuid",
  "userId": "uuid",
  "createdAt": "November 5, 2024T10:00:00Z",
  "lastUsedAt": "November 5, 2024T11:30:00Z",
  "userAgent": "Mozilla/5.0...",
  "ipAddress": "192.168.1.1",
  "country": "US"
}
```

**Session management UI**:
- User can view all active sessions
- User can revoke individual sessions
- User can revoke all sessions (logout everywhere)

### Password Reset

**Flow**:
```
1. User requests password reset
   ↓
2. Generate reset token (random, 32 bytes, hex)
   Store: redis(`reset:${userId}`, token, TTL: 1h)
   ↓
3. Send email with reset link
   URL: https://blog.com/reset-password?token=...
   ↓
4. User clicks link, submits new password
   ↓
5. Validate:
   - Token exists and matches
   - Token not expired
   - New password meets requirements
   - New password != old password
   ↓
6. Update password, revoke all sessions
   Send confirmation email
```

**Security controls**:
- Reset tokens single-use
- 1-hour expiration
- Invalidate old tokens on password change
- Rate limit reset requests (3 per hour per email)
- Email sent even if account doesn't exist (prevent enumeration)

---

## Monitoring & Alerting

### Failed Authentication Monitoring

**Alert triggers**:

| Event | Threshold | Action |
|-------|-----------|--------|
| Failed logins (single IP) | 50/hour | Rate limit IP |
| Failed logins (single account) | 5/15min | Lock account |
| Account lockouts | 10/hour | Security team notified |
| Password reset requests | 100/hour | Investigate potential attack |
| MFA failures (single account) | 10/hour | Lock account, notify user |

**Logging requirements**:
```json
{
  "event": "login_failed",
  "timestamp": "November 5, 2024T10:00:00Z",
  "userId": "uuid or null",
  "email": "hashed",
  "ipAddress": "192.168.1.1",
  "userAgent": "Mozilla/5.0...",
  "reason": "invalid_password|account_locked|mfa_failed",
  "attemptNumber": 3
}
```

**Log all authentication events**:
- ✅ Login attempts (success and failure)
- ✅ Password changes
- ✅ Password reset requests
- ✅ Account lockouts/unlocks
- ✅ MFA enable/disable
- ✅ OAuth connections/disconnections
- ✅ Session creations/revocations

**Log retention**: 2 years (SOC2 requirement)

### Anomaly Detection

**Detect suspicious activity** (future enhancement):
- Login from new country
- Login from new device
- Unusual access time (3 AM when user typically logs in at 9 AM)
- Velocity attack (many accounts from same IP)

**Action**: Email notification, optional step-up authentication

---

## Compliance

### SOC2 Requirements

**CC6.1 - Logical Access Controls**:
- ✅ Unique user accounts (no shared accounts)
- ✅ Strong password requirements
- ✅ MFA for administrative access
- ✅ Session timeouts (1h access token, 7d refresh token)

**CC6.2 - User Registration/Deprovisioning**:
- ✅ Formal user registration process
- ✅ Email verification required
- ✅ Account deactivation workflow
- ✅ Data deletion on account deletion (30-day grace period)

**Evidence for audit**:
- Authentication logs (CloudWatch)
- Password policy documentation (this doc)
- MFA enrollment reports
- Session management logs

---

## Testing

### Security Testing

**Required tests**:
- [ ] Brute force protection (account lockout after 5 attempts)
- [ ] Token expiration (access token after 1h, refresh after 7d)
- [ ] Token revocation (logout invalidates tokens)
- [ ] Password reset (token single-use, 1h expiration)
- [ ] MFA bypass attempt (cannot skip MFA when enabled)
- [ ] OAuth state validation (CSRF prevention)
- [ ] Session limit enforcement (max 5 concurrent)

**Penetration testing**: Annual (Q4)

---

## References

**Related ADRs**:
- [ADR-003: JWT Authentication](../decisions/adr-003-jwt-auth.md)

**Related docs**:
- [Authorization (RBAC)](authorization.md)
- [API Security](api-security.md)
- [Data Protection](data-protection.md)

**Implementation**:
- [Auth Service](../architecture/services/auth-service.md)
- [API Endpoints](../../reference/api/endpoints.md#authentication)

**External standards**:
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [NIST Digital Identity Guidelines](https://pages.nist.gov/800-63-3/)
- [RFC 6749: OAuth 2.0](https://tools.ietf.org/html/rfc6749)
- [RFC 6238: TOTP](https://tools.ietf.org/html/rfc6238)

---

**Maintained By**: Security Team  
**Review Cycle**: Quarterly  
**Last Reviewed**: November 5, 2024  
**Next Review**: February 5, 2025
