# System Architecture Reference

> **Document Type**: Reference (Information-Oriented)  
> **Category**: Architecture  
> **Version**: 1.0  
> **Last Updated**: November 5, 2024

## Overview

Complete technical architecture specification for the Blog Engine system.

**Architecture Style**: Microservices  
**Deployment**: Kubernetes on AWS  
**Primary Language**: Node.js 18+ (TypeScript)

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                          │
├─────────────────────────────────────────────────────────────┤
│  Web App (React)  │  iOS App (RN)  │  Android App (RN)      │
└─────────────────┬───────────────────┬───────────────────────┘
                  │                   │
                  └──────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │   Load Balancer │
                    │   (AWS ALB)     │
                    └────────┬────────┘
                             │
              ┌──────────────▼──────────────┐
              │      API GATEWAY            │
              │   (Express + JWT Auth)      │
              │   Port: 3000                │
              └──────────────┬──────────────┘
                             │
        ┏━━━━━━━━━━━━━━━━━━━━┻━━━━━━━━━━━━━━━━━━━━┓
        ┃          SERVICE LAYER                    ┃
        ┗━━━━━━━━━━━━━━━━━━━━┯━━━━━━━━━━━━━━━━━━━━┛
                             │
    ┌────────┬───────┬───────┼───────┬────────┐
    │        │       │       │       │        │
┌───▼───┐┌──▼───┐┌──▼───┐┌──▼───┐┌──▼────┐┌─▼─────┐
│ Auth  ││ Post ││Comment││User  ││Notify ││ Admin │
│Service││Service│Service││Service│Service││Service│
│ :3001 ││:3002 ││:3003 ││:3004 ││:3005 ││:3006 │
└───┬───┘└──┬───┘└──┬───┘└──┬───┘└──┬────┘└─┬─────┘
    │       │       │       │       │       │
    └───────┴───────┴───────┴───────┴───────┘
                    │
       ┌────────────┼────────────┐
       │            │            │
┌──────▼──────┐ ┌──▼──────┐ ┌──▼────────┐
│ PostgreSQL  │ │  Redis  │ │ RabbitMQ  │
│ (Primary DB)│ │ (Cache) │ │ (Events)  │
└─────────────┘ └─────────┘ └───────────┘
```

---

## Service Responsibilities

### API Gateway (Port 3000)

**Purpose**: Single entry point for all client requests

**Responsibilities**:
- Request routing to appropriate services
- JWT token validation
- Rate limiting (100 req/min per IP)
- Request/response logging
- CORS handling

**Technology**: Express.js, express-jwt, express-rate-limit

**Endpoints**: Routes all `/api/*` to backend services

---

### Auth Service (Port 3001)

**Purpose**: User authentication and authorization

**Responsibilities**:
- User registration and login
- JWT token generation and refresh
- OAuth2 integration (Google, GitHub)
- Password reset flow
- Session management

**Database Tables**:
- `users`
- `sessions`
- `oauth_tokens`

**APIs**:
- `POST /auth/register`
- `POST /auth/login`
- `POST /auth/refresh`
- `POST /auth/logout`
- `GET /auth/oauth/:provider`

**Dependencies**: PostgreSQL, Redis (session cache)

---

### Post Service (Port 3002)

**Purpose**: Blog post CRUD operations

**Responsibilities**:
- Create, read, update, delete posts
- Draft/publish workflow
- Image upload to S3/CDN
- Full-text search (PostgreSQL)
- Post analytics (views, likes)

**Database Tables**:
- `posts`
- `post_tags`
- `post_likes`

**APIs**:
- `GET /posts` - List posts (paginated)
- `GET /posts/:id` - Get single post
- `POST /posts` - Create post (auth required)
- `PUT /posts/:id` - Update post (author only)
- `DELETE /posts/:id` - Delete post (author/admin)
- `POST /posts/:id/like` - Like post

**Events Published**:
- `post.created`
- `post.published`
- `post.updated`
- `post.deleted`

**Dependencies**: PostgreSQL, Redis, RabbitMQ, AWS S3

---

### Comment Service (Port 3003)

**Purpose**: Comment system with moderation

**Responsibilities**:
- Add/edit/delete comments
- Threaded replies (parent-child)
- Comment moderation queue
- Spam detection
- Like/flag comments

**Database Tables**:
- `comments`
- `comment_likes`

**APIs**:
- `GET /posts/:postId/comments` - List comments
- `POST /posts/:postId/comments` - Add comment
- `PUT /comments/:id` - Edit comment
- `DELETE /comments/:id` - Delete comment
- `POST /comments/:id/like` - Like comment
- `POST /comments/:id/flag` - Flag for moderation

**Events Published**:
- `comment.created`
- `comment.flagged`

**Dependencies**: PostgreSQL, RabbitMQ

---

### User Service (Port 3004)

**Purpose**: User profile management

**Responsibilities**:
- User profile CRUD
- Avatar upload
- Following/followers
- User preferences
- Activity feed

**Database Tables**:
- `users` (shared with Auth)
- `user_follows`
- `user_preferences`

**APIs**:
- `GET /users/:id` - Get user profile
- `PUT /users/:id` - Update profile
- `GET /users/:id/posts` - User's posts
- `POST /users/:id/follow` - Follow user
- `GET /users/:id/followers` - List followers

**Dependencies**: PostgreSQL, Redis

---

### Notification Service (Port 3005)

**Purpose**: Multi-channel notifications

**Responsibilities**:
- Email notifications (SendGrid)
- Push notifications (Firebase)
- In-app notifications
- Notification preferences
- Delivery status tracking

**Database Tables**:
- `notifications`
- `notification_preferences`

**APIs**:
- `GET /notifications` - List user notifications
- `PUT /notifications/:id/read` - Mark as read
- `PUT /notifications/settings` - Update preferences

**Events Consumed**:
- `post.created` → notify followers
- `comment.created` → notify post author
- `post.liked` → notify author
- `user.followed` → notify user

**External Services**:
- SendGrid (email)
- Firebase Cloud Messaging (push)

**Dependencies**: PostgreSQL, RabbitMQ, SendGrid API, Firebase

---

### Admin Service (Port 3006)

**Purpose**: Admin panel operations

**Responsibilities**:
- Content moderation
- User management (suspend/ban)
- Analytics dashboard
- System health monitoring
- Configuration management

**APIs**:
- `GET /admin/posts/flagged` - Moderation queue
- `POST /admin/posts/:id/approve` - Approve post
- `POST /admin/users/:id/suspend` - Suspend user
- `GET /admin/analytics` - System metrics

**Authorization**: Requires admin role in JWT

**Dependencies**: PostgreSQL, Redis

---

## Data Layer

### PostgreSQL (Primary Database)

**Version**: 14+  
**Deployment**: AWS RDS Multi-AZ  
**Connection Pool**: 20-100 connections per service

**Databases**:
- `blogengine_prod` (production)
- `blogengine_staging` (staging)
- `blogengine_dev` (local development)

**Key Tables**: See [Database Schema](../database/schema.md)

**Backup Strategy**:
- Automated daily backups (7-day retention)
- Point-in-time recovery enabled
- Manual snapshots before major deployments

---

### Redis (Cache + Sessions)

**Version**: 7+  
**Deployment**: AWS ElastiCache (Redis)  
**Mode**: Cluster mode

**Usage**:
- Session storage (Auth Service)
- API response caching (5-min TTL)
- Rate limiting counters
- Real-time data (online users)

**Cache Keys**:
- `session:{userId}:{tokenId}` - User sessions
- `post:{postId}` - Cached post data
- `user:{userId}` - Cached user profiles
- `ratelimit:{ip}:{endpoint}` - Rate limit counters

**Eviction Policy**: LRU (Least Recently Used)

---

### RabbitMQ (Message Queue)

**Version**: 3.x  
**Deployment**: AWS MQ (RabbitMQ)

**Exchanges**:
- `blog.events` (topic) - All domain events
- `blog.notifications` (direct) - Notification events

**Queues**:
- `notification.email` - Email processing
- `notification.push` - Push notification processing
- `analytics.events` - Analytics processing
- `search.indexing` - Search index updates

**Message Format**: JSON with schema validation

---

## Cross-Cutting Concerns

### Authentication Flow

1. User sends credentials to API Gateway
2. Gateway forwards to Auth Service
3. Auth Service validates and generates JWT
4. JWT contains: `{userId, role, exp}`
5. All subsequent requests include JWT in header
6. Each service validates JWT independently (shared secret)

---

### Service Communication

**Synchronous**: HTTP/REST between services (rare)  
**Asynchronous**: RabbitMQ events (primary pattern)

**Event Pattern**:
```javascript
{
  "eventType": "post.created",
  "eventId": "uuid",
  "timestamp": "November 5, 2024T10:00:00Z",
  "payload": {
    "postId": "uuid",
    "authorId": "uuid",
    "title": "Post title"
  }
}
```

---

### Error Handling

All services return standard error format:
```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message",
    "details": {},
    "requestId": "uuid"
  }
}
```

---

### Logging

**Framework**: Winston  
**Format**: JSON structured logs  
**Destination**: CloudWatch Logs

**Log Levels**:
- ERROR: Critical issues requiring immediate attention
- WARN: Degraded functionality
- INFO: Normal operations
- DEBUG: Detailed diagnostic info

---

### Monitoring

**Metrics**: Prometheus + Grafana  
**Traces**: Jaeger (distributed tracing)  
**Alerts**: PagerDuty integration

**Key Metrics**:
- Request rate, latency, error rate (per service)
- Database connection pool usage
- Queue depth and processing time
- Cache hit/miss ratio

---

## Deployment Architecture

### Kubernetes Cluster

**Provider**: AWS EKS  
**Node Size**: t3.medium (2 vCPU, 4GB RAM)  
**Auto-scaling**: 3-10 nodes

**Namespaces**:
- `production`
- `staging`
- `monitoring`

**Services per Pod**:
- 2-5 replicas per service
- Resource limits: 500m CPU, 512Mi memory
- Liveness and readiness probes

---

### CI/CD Pipeline

**Tool**: GitHub Actions

**Flow**:
1. PR created → Run tests
2. Merge to main → Build Docker images
3. Push to ECR
4. Deploy to staging
5. Manual approval
6. Deploy to production (rolling update)

---

### Scalability Considerations

**Horizontal Scaling**:
- All services are stateless
- Database connections pooled
- Shared cache layer

**Bottlenecks**:
- PostgreSQL: Use read replicas for queries
- RabbitMQ: Cluster mode for high throughput
- API Gateway: Multiple instances behind load balancer

**Target Capacity**:
- 1,000 concurrent users
- 10,000 requests/minute
- 100,000 posts
- 1,000,000 comments

---

## Security Architecture

**Network**: VPC with private subnets for services  
**Database**: Not publicly accessible  
**Secrets**: AWS Secrets Manager  
**SSL/TLS**: Certificate Manager (ACM)

**API Security**:
- JWT authentication
- Rate limiting per IP/user
- Input validation and sanitization
- SQL injection protection (parameterized queries)
- XSS protection (Content Security Policy)

---

## Related Documentation

- [Database Schema](../database/schema.md)
- [API Endpoints](../api/endpoints.md)
- [ADR-001: Microservices Architecture](../../docs/decisions/adr-001-microservices.md)
- [Deploy to Production](../../how-to/deployment/deploy-to-production.md)

---

**Maintained By**: Architecture Team  
**Last Reviewed**: November 5, 2024  
**Review Cycle**: Quarterly
