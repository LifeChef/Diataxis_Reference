# Service Catalog

> **Document Type**: Reference (SAMPLE)  
> **Audience**: Developers, Architects, DevOps  
> **Last Updated**: November 5, 2024

---

## Overview

This catalog documents all microservices in the Blog Engine system, including their responsibilities, APIs, dependencies, and deployment information.

---

## Service Architecture

```
                    ┌─────────────────┐
                    │   API Gateway   │
                    │    (Port 3000)  │
                    └────────┬────────┘
                             │
         ┌───────────────────┼───────────────────┬────────────┐
         │                   │                   │            │
    ┌────▼────┐      ┌──────▼──────┐     ┌─────▼─────┐  ┌──▼────────┐
    │  Auth   │      │    Post     │     │  Comment  │  │  Notify   │
    │ Service │      │   Service   │     │  Service  │  │  Service  │
    │ :3001   │      │    :3002    │     │   :3003   │  │   :3005   │
    └─────────┘      └─────────────┘     └───────────┘  └───────────┘
```

---

## Service Directory

### Core Services

| Service | Port | Status | Owner | Repository |
|---------|------|--------|-------|------------|
| [API Gateway](#api-gateway) | 3000 | ✅ Production | Platform Team | `blog-engine/api-gateway` |
| [Auth Service](#auth-service) | 3001 | ✅ Production | Security Team | `blog-engine/auth-service` |
| [Post Service](#post-service) | 3002 | ✅ Production | Content Team | `blog-engine/post-service` |
| [Comment Service](#comment-service) | 3003 | ✅ Production | Content Team | `blog-engine/comment-service` |
| [User Service](#user-service) | 3004 | ✅ Production | Platform Team | `blog-engine/user-service` |
| [Notification Service](#notification-service) | 3005 | ✅ Production | Platform Team | `blog-engine/notification-service` |
| [Admin Service](#admin-service) | 3006 | 🔄 Beta | Admin Team | `blog-engine/admin-service` |

---

## API Gateway

**Purpose**: Single entry point for all client requests, handles routing, authentication, rate limiting

**Responsibilities**:
- Request routing to appropriate microservices
- JWT token validation
- Rate limiting and throttling
- Request/response logging
- API versioning
- CORS handling

**Technology**: Node.js, Express, Redis

**Key Dependencies**:
- Redis (rate limiting, sessions)
- All backend services

**Endpoints**:
- `GET /health` - Health check
- `/*` - Proxy to microservices

**Configuration**:
```yaml
PORT: 3000
REDIS_URL: redis://redis:6379
RATE_LIMIT: 100 requests/minute
JWT_SECRET: ${JWT_SECRET}
```

**Deployment**:
- Replicas: 3
- CPU: 500m
- Memory: 512Mi

---

## Auth Service

**Purpose**: User authentication, authorization, and session management

**Responsibilities**:
- User registration and login
- JWT token generation and validation
- OAuth2 integration (Google, GitHub)
- Password hashing (bcrypt)
- Multi-factor authentication (MFA)
- Session management
- Role-based access control (RBAC)

**Technology**: Node.js, Express, PostgreSQL, Redis

**Database Tables**:
- `users`
- `sessions`
- `oauth_tokens`
- `user_roles`

**Key Endpoints**:
- `POST /auth/register` - User registration
- `POST /auth/login` - User login
- `POST /auth/logout` - User logout
- `POST /auth/refresh` - Refresh JWT token
- `GET /auth/me` - Get current user
- `POST /auth/oauth/google` - Google OAuth

**Events Published**:
- `user.registered` - New user signed up
- `user.logged_in` - User logged in
- `user.logged_out` - User logged out

**Configuration**:
```yaml
PORT: 3001
DATABASE_URL: ${DATABASE_URL}
REDIS_URL: ${REDIS_URL}
JWT_SECRET: ${JWT_SECRET}
JWT_EXPIRY: 1h
BCRYPT_ROUNDS: 10
GOOGLE_CLIENT_ID: ${GOOGLE_CLIENT_ID}
GOOGLE_CLIENT_SECRET: ${GOOGLE_CLIENT_SECRET}
```

---

## Post Service

**Purpose**: Blog post creation, retrieval, and management

**Responsibilities**:
- CRUD operations for blog posts
- Draft/publish workflow
- Tag management
- Image upload to S3
- Post search and filtering
- View counting
- Post likes/reactions

**Technology**: Node.js, Express, PostgreSQL, S3, Elasticsearch

**Database Tables**:
- `posts`
- `post_tags`
- `post_likes`
- `post_views`

**Key Endpoints**:
- `GET /posts` - List posts (paginated)
- `GET /posts/:id` - Get single post
- `POST /posts` - Create post (auth required)
- `PUT /posts/:id` - Update post (owner/admin only)
- `DELETE /posts/:id` - Delete post (owner/admin only)
- `POST /posts/:id/like` - Like post
- `POST /posts/:id/upload` - Upload image

**Events Published**:
- `post.created` - New post published
- `post.updated` - Post edited
- `post.deleted` - Post removed

**Events Consumed**:
- `user.deleted` - Delete user's posts

**Configuration**:
```yaml
PORT: 3002
DATABASE_URL: ${DATABASE_URL}
AWS_S3_BUCKET: ${AWS_S3_BUCKET}
AWS_ACCESS_KEY: ${AWS_ACCESS_KEY}
AWS_SECRET_KEY: ${AWS_SECRET_KEY}
CDN_URL: ${CDN_URL}
ELASTICSEARCH_URL: ${ELASTICSEARCH_URL}
```

---

## Comment Service

**Purpose**: Comment management and moderation

**Responsibilities**:
- CRUD operations for comments
- Nested comment threads
- Comment moderation queue
- Spam detection
- Comment likes
- User mention notifications

**Technology**: Node.js, Express, PostgreSQL

**Database Tables**:
- `comments`
- `comment_likes`
- `moderation_queue`

**Key Endpoints**:
- `GET /posts/:postId/comments` - Get post comments
- `POST /posts/:postId/comments` - Create comment
- `PUT /comments/:id` - Edit comment
- `DELETE /comments/:id` - Delete comment
- `POST /comments/:id/like` - Like comment

**Events Published**:
- `comment.created` - New comment added
- `comment.flagged` - Comment reported

**Events Consumed**:
- `post.deleted` - Delete associated comments
- `user.deleted` - Delete user's comments

---

## User Service

**Purpose**: User profile and preferences management

**Responsibilities**:
- User profile CRUD
- Avatar upload
- User preferences
- Following/followers
- User activity tracking

**Technology**: Node.js, Express, PostgreSQL, S3

**Database Tables**:
- `user_profiles`
- `user_preferences`
- `user_follows`
- `user_activity`

**Key Endpoints**:
- `GET /users/:id` - Get user profile
- `PUT /users/:id` - Update profile
- `POST /users/:id/follow` - Follow user
- `DELETE /users/:id/follow` - Unfollow user
- `GET /users/:id/followers` - Get followers

---

## Notification Service

**Purpose**: Multi-channel notifications (email, push, in-app)

**Responsibilities**:
- Send email notifications (SendGrid)
- Send push notifications (Firebase)
- In-app notification management
- Notification preferences
- Event-driven notification triggers

**Technology**: Node.js, Express, PostgreSQL, RabbitMQ, SendGrid, Firebase

**Database Tables**:
- `notifications`
- `notification_preferences`
- `email_queue`

**Key Endpoints**:
- `GET /notifications` - Get user notifications
- `PUT /notifications/:id/read` - Mark as read
- `PUT /notifications/preferences` - Update preferences

**Events Consumed**:
- `post.created` - Notify followers
- `comment.created` - Notify post author
- `user.mentioned` - Notify mentioned user

**Configuration**:
```yaml
PORT: 3005
SENDGRID_API_KEY: ${SENDGRID_API_KEY}
FIREBASE_SERVER_KEY: ${FIREBASE_SERVER_KEY}
RABBITMQ_URL: ${RABBITMQ_URL}
```

---

## Admin Service

**Purpose**: Admin panel operations and content moderation

**Responsibilities**:
- Content moderation
- User management
- Analytics and reporting
- System configuration
- Audit logging

**Technology**: Node.js, Express, PostgreSQL

**Key Endpoints**:
- `GET /admin/stats` - Dashboard statistics
- `GET /admin/moderation/queue` - Moderation queue
- `PUT /admin/posts/:id/moderate` - Moderate content
- `PUT /admin/users/:id/suspend` - Suspend user

---

## Cross-Cutting Concerns

### Logging
All services use structured JSON logging:
```json
{
  "timestamp": "November 5, 2024T14:30:00Z",
  "level": "info",
  "service": "post-service",
  "message": "Post created",
  "userId": 123,
  "postId": 456,
  "correlationId": "abc-123"
}
```

### Monitoring
- Prometheus metrics exposed at `/metrics`
- Health checks at `/health`
- Distributed tracing with Jaeger

### Error Handling
Standardized error responses:
```json
{
  "error": {
    "code": "POST_NOT_FOUND",
    "message": "Post with ID 123 not found",
    "status": 404
  }
}
```

---

## Related Documentation

- [ADR-001: Microservices Architecture](../../decisions/adr-001-microservices.md)
- [System Architecture Overview](../system-overview.md)
- [API Reference](../../api/rest-api.md)
- [Deployment Guide](../../../how-to/deployment/deploy-to-production.md)

---

**Maintained By**: Architecture Team  
**Review Frequency**: Monthly  
**Next Review**: December 5, 2024
