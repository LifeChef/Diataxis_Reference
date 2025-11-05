# ADR-003: JWT for API Authentication

**Status**: ✅ Accepted  
**Date**: November 5, 2024  
**Deciders**: Architecture Team, Security Team  
**Related**: ADR-001 (Microservices), Auth Service, Security  
**Supersedes**: -

---

## Context

Need authentication mechanism for API requests across microservices. Requirements:

1. **Stateless**: Services shouldn't store session state (microservices principle)
2. **Cross-service**: Auth token must work across all services
3. **Standard**: Use industry-standard approach
4. **Secure**: Protect against common attacks (CSRF, XSS, token theft)
5. **Performance**: Minimal latency overhead (<5ms verification)
6. **Revocation**: Ability to invalidate tokens when needed
7. **OAuth2 integration**: Support Google/GitHub OAuth

---

## Decision

Use **JWT (JSON Web Tokens)** with **RS256** (RSA public/private key signing) for API authentication.

**Token structure**:
```json
{
  "userId": "uuid",
  "email": "user@example.com",
  "role": "user|admin",
  "iat": 1699200000,
  "exp": 1699203600
}
```

**Token types**:
- **Access token**: Short-lived (1 hour), used for API requests
- **Refresh token**: Long-lived (7 days), used to get new access tokens

**Verification**:
- API Gateway verifies JWT signature using public key
- Each service can also verify independently (defense in depth)
- Invalid/expired tokens → 401 Unauthorized

**Storage**:
- Access token: Memory only (client-side)
- Refresh token: Redis (server-side) + HttpOnly cookie (client-side)

---

## Consequences

### Positive

1. **Stateless**: No server-side session storage needed for access tokens
2. **Scalable**: Works seamlessly across microservices
3. **Fast verification**: Public key verification ~2ms per request
4. **Industry standard**: Well-understood, extensive library support
5. **Self-contained**: Token contains all needed info (user ID, role)
6. **OAuth2 compatible**: Works with Google/GitHub OAuth flows

### Negative

1. **Cannot revoke individual access tokens**: Must wait for expiration (1h)
   - *Mitigation*: Short TTL (1h) + refresh token revocation in Redis
   
2. **Token size**: ~500 bytes vs 32-byte session ID
   - *Mitigation*: Acceptable overhead, compressed in HTTP/2

3. **Clock synchronization**: Requires NTP for exp validation
   - *Mitigation*: Use NTP on all servers (standard practice)

### Neutral

- Need key rotation strategy (quarterly recommended)
- Must protect private key (AWS Secrets Manager)

---

## Alternatives Considered

### Alternative 1: Session Cookies (Server-side sessions)
**Description**: Traditional session ID in cookie, state in Redis  
**Pros**: Can revoke immediately, smaller cookie size  
**Cons**: Requires sticky sessions or shared Redis, not truly stateless  
**Why Not**: Breaks microservices stateless principle. Redis becomes single point of failure.

### Alternative 2: OAuth2 Only (No custom auth)
**Description**: Force users to use Google/GitHub OAuth only  
**Pros**: No password management, standard protocol  
**Cons**: Requires external provider, some users prefer email/password  
**Why Not**: Want to support both OAuth and custom auth for flexibility.

### Alternative 3: API Keys (Long-lived tokens)
**Description**: Issue API keys for authentication  
**Pros**: Simple, works for service-to-service  
**Cons**: Hard to revoke, no user context, not suitable for user auth  
**Why Not**: Need user-specific authentication, not just API access.

---

## Implementation

**JWT Configuration**:
```javascript
// Auth Service generates tokens
{
  algorithm: 'RS256',
  expiresIn: '1h',           // Access token
  issuer: 'blog-engine',
  audience: 'blog-engine-api'
}

// Refresh token
{
  expiresIn: '7d',
  stored: 'redis',             // Can revoke
  key: `refresh:${userId}:${tokenId}`
}
```

**Key management**:
- RS256 public/private key pair
- Private key in AWS Secrets Manager
- Public key distributed to all services
- Rotate keys quarterly

**Revocation**:
- Maintain blocklist in Redis for emergency revocation
- Check blocklist on critical operations (password change, etc.)
- Refresh token revocation: Delete from Redis

**Affected services**: All

---

## References

**Implementation**:
- [Authentication Security](../security/authentication.md)
- [Auth Service](../architecture/services/auth-service.md)

**Related ADRs**:
- [ADR-001: Microservices](adr-001-microservices.md)
- [ADR-005: Redis for Caching](adr-005-redis-cache.md)

**Features**:
- [User Authentication](../../features/user-authentication/)

**External**:
- [JWT.io](https://jwt.io/)
- [RFC 7519: JWT](https://tools.ietf.org/html/rfc7519)
- [OWASP JWT Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)

---

## Notes

**Security considerations**:
- Never log JWT tokens (contain sensitive data)
- Always use HTTPS (prevent token interception)
- Validate `exp`, `iss`, `aud` claims
- Use `HttpOnly` cookies for refresh tokens (prevent XSS)

**Watch for**:
- Token size growth (keep payload minimal)
- Key rotation schedule (set calendar reminder)
- Refresh token abuse (rate limit refresh endpoint)

**Reconsider if**:
- Need instant token revocation (consider session-based)
- Token size becomes problematic (>1KB)
- Clock skew issues arise (consider allowing 30s leeway)

**Success metrics**:
- Token verification latency: <5ms P95
- Failed auth rate: <2% (indicates attacks)
- Key rotation: On schedule (quarterly)

**Review schedule**: Annual security audit

---

**Author**: Security Team + Backend Team  
**Reviewers**: CTO, Security Lead  
**Last Updated**: November 5, 2024
