# Configuration Parameters Reference

> **Document Type**: Reference (Information-Oriented)  
> **Category**: Configuration  
> **Version**: 1.0  
> **Last Updated**: November 5, 2024

## Overview

Complete reference for all environment variables and configuration parameters used across Blog Engine services.

**Configuration Method**: Environment variables  
**Format**: `.env` files in each service directory  
**Validation**: Checked at service startup

---

## Common Parameters (All Services)

Required by all microservices.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `NODE_ENV` | string | Yes | development | Environment: `development`, `staging`, `production` |
| `PORT` | integer | Yes | (varies) | Service HTTP port |
| `LOG_LEVEL` | string | No | info | Logging level: `error`, `warn`, `info`, `debug` |
| `SERVICE_NAME` | string | Yes | - | Service identifier for logging/tracing |

**Example**:
```bash
NODE_ENV=production
PORT=3001
LOG_LEVEL=info
SERVICE_NAME=auth-service
```

---

## Database Configuration

PostgreSQL connection parameters.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `DB_HOST` | string | Yes | localhost | PostgreSQL host |
| `DB_PORT` | integer | No | 5432 | PostgreSQL port |
| `DB_NAME` | string | Yes | - | Database name |
| `DB_USER` | string | Yes | - | Database user |
| `DB_PASSWORD` | string | Yes | - | Database password |
| `DB_SSL` | boolean | No | false | Enable SSL connection |
| `DB_POOL_MIN` | integer | No | 10 | Minimum connections in pool |
| `DB_POOL_MAX` | integer | No | 100 | Maximum connections in pool |
| `DB_IDLE_TIMEOUT` | integer | No | 30000 | Idle timeout (ms) |

**Example**:
```bash
DB_HOST=blogengine-prod.abc123.us-east-1.rds.amazonaws.com
DB_PORT=5432
DB_NAME=blogengine_prod
DB_USER=blog_app
DB_PASSWORD=<from secrets manager>
DB_SSL=true
DB_POOL_MIN=20
DB_POOL_MAX=100
```

---

## Redis Configuration

Cache and session storage parameters.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `REDIS_HOST` | string | Yes | localhost | Redis host |
| `REDIS_PORT` | integer | No | 6379 | Redis port |
| `REDIS_PASSWORD` | string | No | - | Redis password (if auth enabled) |
| `REDIS_DB` | integer | No | 0 | Redis database number |
| `REDIS_TLS` | boolean | No | false | Enable TLS connection |
| `REDIS_TTL` | integer | No | 300 | Default cache TTL (seconds) |

**Example**:
```bash
REDIS_HOST=blogengine-cache.abc123.cache.amazonaws.com
REDIS_PORT=6379
REDIS_PASSWORD=<from secrets manager>
REDIS_TLS=true
REDIS_TTL=300
```

---

## RabbitMQ Configuration

Message queue parameters.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `RABBITMQ_URL` | string | Yes | - | Full connection URL |
| `RABBITMQ_EXCHANGE` | string | No | blog.events | Exchange name |
| `RABBITMQ_PREFETCH` | integer | No | 10 | Messages to prefetch |
| `RABBITMQ_RECONNECT_DELAY` | integer | No | 5000 | Reconnect delay (ms) |

**Example**:
```bash
RABBITMQ_URL=amqps://user:pass@broker.example.com:5671/vhost
RABBITMQ_EXCHANGE=blog.events
RABBITMQ_PREFETCH=10
```

---

## Authentication Configuration (Auth Service)

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `JWT_SECRET` | string | Yes | - | Secret for signing JWT tokens |
| `JWT_EXPIRATION` | string | No | 1h | Access token expiration |
| `JWT_REFRESH_EXPIRATION` | string | No | 7d | Refresh token expiration |
| `BCRYPT_ROUNDS` | integer | No | 10 | Password hashing rounds |
| `OAUTH_GOOGLE_CLIENT_ID` | string | No | - | Google OAuth client ID |
| `OAUTH_GOOGLE_CLIENT_SECRET` | string | No | - | Google OAuth secret |
| `OAUTH_GITHUB_CLIENT_ID` | string | No | - | GitHub OAuth client ID |
| `OAUTH_GITHUB_CLIENT_SECRET` | string | No | - | GitHub OAuth secret |

**Example**:
```bash
JWT_SECRET=<generate-random-256-bit-key>
JWT_EXPIRATION=1h
JWT_REFRESH_EXPIRATION=7d
BCRYPT_ROUNDS=12
OAUTH_GOOGLE_CLIENT_ID=abc123.apps.googleusercontent.com
OAUTH_GOOGLE_CLIENT_SECRET=<from secrets>
```

**⚠️ Security Note**: Never commit `JWT_SECRET` or OAuth secrets to git. Use AWS Secrets Manager or similar.

---

## API Gateway Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `GATEWAY_PORT` | integer | No | 3000 | Gateway HTTP port |
| `AUTH_SERVICE_URL` | string | Yes | - | Auth service base URL |
| `POST_SERVICE_URL` | string | Yes | - | Post service base URL |
| `COMMENT_SERVICE_URL` | string | Yes | - | Comment service base URL |
| `USER_SERVICE_URL` | string | Yes | - | User service base URL |
| `NOTIFY_SERVICE_URL` | string | Yes | - | Notification service base URL |
| `ADMIN_SERVICE_URL` | string | Yes | - | Admin service base URL |
| `RATE_LIMIT_WINDOW` | integer | No | 60000 | Rate limit window (ms) |
| `RATE_LIMIT_MAX` | integer | No | 100 | Max requests per window |
| `CORS_ORIGIN` | string | Yes | - | Allowed CORS origins (comma-separated) |

**Example**:
```bash
GATEWAY_PORT=3000
AUTH_SERVICE_URL=http://auth-service:3001
POST_SERVICE_URL=http://post-service:3002
COMMENT_SERVICE_URL=http://comment-service:3003
USER_SERVICE_URL=http://user-service:3004
NOTIFY_SERVICE_URL=http://notify-service:3005
ADMIN_SERVICE_URL=http://admin-service:3006
RATE_LIMIT_WINDOW=60000
RATE_LIMIT_MAX=100
CORS_ORIGIN=https://blog.example.com,https://admin.blog.example.com
```

---

## Post Service Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `AWS_S3_BUCKET` | string | Yes | - | S3 bucket for uploads |
| `AWS_S3_REGION` | string | Yes | - | S3 bucket region |
| `AWS_ACCESS_KEY_ID` | string | Yes | - | AWS access key |
| `AWS_SECRET_ACCESS_KEY` | string | Yes | - | AWS secret key |
| `CDN_URL` | string | No | - | CloudFront CDN URL |
| `MAX_UPLOAD_SIZE` | integer | No | 5242880 | Max file size (bytes, default 5MB) |
| `ALLOWED_MIME_TYPES` | string | No | image/jpeg,image/png,image/gif | Allowed upload types |

**Example**:
```bash
AWS_S3_BUCKET=blogengine-uploads-prod
AWS_S3_REGION=us-east-1
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=<from secrets>
CDN_URL=https://d1234567890.cloudfront.net
MAX_UPLOAD_SIZE=5242880
ALLOWED_MIME_TYPES=image/jpeg,image/png,image/gif,image/webp
```

---

## Notification Service Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `SENDGRID_API_KEY` | string | Yes | - | SendGrid API key |
| `SENDGRID_FROM_EMAIL` | string | Yes | - | Sender email address |
| `SENDGRID_FROM_NAME` | string | No | Blog Engine | Sender name |
| `FIREBASE_PROJECT_ID` | string | No | - | Firebase project ID |
| `FIREBASE_PRIVATE_KEY` | string | No | - | Firebase service account key |
| `FIREBASE_CLIENT_EMAIL` | string | No | - | Firebase service account email |
| `NOTIFICATION_BATCH_SIZE` | integer | No | 100 | Max notifications per batch |
| `NOTIFICATION_RETRY_ATTEMPTS` | integer | No | 3 | Retry failed sends |

**Example**:
```bash
SENDGRID_API_KEY=SG.abc123...
SENDGRID_FROM_EMAIL=noreply@blog.example.com
SENDGRID_FROM_NAME=Blog Engine
FIREBASE_PROJECT_ID=blogengine-prod
FIREBASE_PRIVATE_KEY=<from secrets>
FIREBASE_CLIENT_EMAIL=firebase-adminsdk@blogengine.iam.gserviceaccount.com
```

---

## Frontend Configuration (Web App)

React environment variables (must start with `REACT_APP_`).

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `REACT_APP_API_URL` | string | Yes | - | API Gateway URL |
| `REACT_APP_WS_URL` | string | No | - | WebSocket URL (future) |
| `REACT_APP_GA_TRACKING_ID` | string | No | - | Google Analytics ID |
| `REACT_APP_SENTRY_DSN` | string | No | - | Sentry error tracking |

**Example**:
```bash
REACT_APP_API_URL=https://api.blog.example.com
REACT_APP_GA_TRACKING_ID=UA-123456789-1
REACT_APP_SENTRY_DSN=https://abc@sentry.io/123456
```

---

## Mobile App Configuration (React Native)

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `API_URL` | string | Yes | - | API Gateway URL |
| `GOOGLE_SERVICES_JSON` | file | Yes (Android) | - | Firebase config |
| `GOOGLE_SERVICE_INFO_PLIST` | file | Yes (iOS) | - | Firebase config |
| `CODEPUSH_KEY_ANDROID` | string | No | - | CodePush deployment key |
| `CODEPUSH_KEY_IOS` | string | No | - | CodePush deployment key |

---

## Admin Service Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `ADMIN_EMAIL` | string | Yes | - | Default admin email |
| `METRICS_RETENTION_DAYS` | integer | No | 90 | Keep metrics for X days |

**Example**:
```bash
ADMIN_EMAIL=admin@blog.example.com
METRICS_RETENTION_DAYS=90
```

---

## Monitoring & Observability

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `PROMETHEUS_PORT` | integer | No | 9090 | Metrics endpoint port |
| `JAEGER_AGENT_HOST` | string | No | - | Jaeger tracing host |
| `JAEGER_AGENT_PORT` | integer | No | 6831 | Jaeger agent port |
| `SENTRY_DSN` | string | No | - | Sentry error tracking |

**Example**:
```bash
PROMETHEUS_PORT=9090
JAEGER_AGENT_HOST=jaeger-agent
JAEGER_AGENT_PORT=6831
SENTRY_DSN=https://abc@sentry.io/123456
```

---

## Environment-Specific Templates

### Local Development (`.env`)

```bash
# Service
NODE_ENV=development
PORT=3001
SERVICE_NAME=auth-service

# Database
DB_HOST=localhost
DB_PORT=5432
DB_NAME=blogengine_dev
DB_USER=postgres
DB_PASSWORD=postgres
DB_POOL_MIN=5
DB_POOL_MAX=20

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379

# RabbitMQ
RABBITMQ_URL=amqp://guest:guest@localhost:5672

# JWT
JWT_SECRET=local-dev-secret-change-in-production
JWT_EXPIRATION=24h
```

### Staging (AWS Secrets Manager)

```bash
# Service
NODE_ENV=staging
PORT=3001
SERVICE_NAME=auth-service

# Database (RDS)
DB_HOST=blogengine-staging.abc123.us-east-1.rds.amazonaws.com
DB_NAME=blogengine_staging
DB_USER=<from secrets>
DB_PASSWORD=<from secrets>
DB_SSL=true

# Redis (ElastiCache)
REDIS_HOST=blogengine-staging.abc123.cache.amazonaws.com
REDIS_TLS=true

# RabbitMQ (Amazon MQ)
RABBITMQ_URL=<from secrets>

# JWT
JWT_SECRET=<from secrets>
```

### Production (AWS Secrets Manager)

```bash
# Service
NODE_ENV=production
PORT=3001
SERVICE_NAME=auth-service
LOG_LEVEL=warn

# Database (RDS Multi-AZ)
DB_HOST=blogengine-prod.abc123.us-east-1.rds.amazonaws.com
DB_NAME=blogengine_prod
DB_USER=<from secrets>
DB_PASSWORD=<from secrets>
DB_SSL=true
DB_POOL_MIN=20
DB_POOL_MAX=100

# Redis (ElastiCache Cluster)
REDIS_HOST=blogengine-prod.abc123.cache.amazonaws.com
REDIS_TLS=true

# RabbitMQ (Amazon MQ Cluster)
RABBITMQ_URL=<from secrets>

# JWT
JWT_SECRET=<from secrets>
JWT_EXPIRATION=1h
BCRYPT_ROUNDS=12

# External Services
SENDGRID_API_KEY=<from secrets>
AWS_ACCESS_KEY_ID=<from secrets>
AWS_SECRET_ACCESS_KEY=<from secrets>
```

---

## Validation

All services validate environment variables on startup. If required variables are missing, the service will not start and will log errors.

**Validation Library**: `envalid` (Node.js)

**Example validation**:
```javascript
const env = require('envalid');

const config = env.cleanEnv(process.env, {
  NODE_ENV: env.str({ choices: ['development', 'staging', 'production'] }),
  PORT: env.port(),
  DB_HOST: env.host(),
  DB_PASSWORD: env.str(),
  JWT_SECRET: env.str({ minLength: 32 }),
});
```

---

## Security Best Practices

1. **Never commit secrets** to version control
2. **Use AWS Secrets Manager** or similar for production
3. **Rotate secrets regularly** (quarterly minimum)
4. **Restrict IAM permissions** to least privilege
5. **Encrypt secrets at rest** in Secrets Manager
6. **Use different secrets per environment**
7. **Audit secret access** via CloudTrail

---

## Troubleshooting

### Problem: Service won't start, "Missing required environment variable"

**Solution**: Check service logs for specific variable name. Add to `.env` file.

### Problem: Database connection fails

**Solution**: 
- Verify `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`
- Check network connectivity: `telnet $DB_HOST $DB_PORT`
- Verify database user has correct permissions

### Problem: JWT tokens invalid

**Solution**:
- Ensure `JWT_SECRET` is identical across all services
- Check token hasn't expired
- Verify clock sync across services (NTP)

---

## Related Documentation

- [How-To: Environment Setup](../../how-to/configuration/environment-setup.md) (planned)
- [System Architecture](../architecture/system-architecture.md)
- [Deploy to Production](../../how-to/deployment/deploy-to-production.md)

---

**Maintained By**: DevOps Team  
**Last Reviewed**: November 5, 2024  
**Review Frequency**: After any new service or integration
