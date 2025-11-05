# Blog Posts - References

> **Purpose**: Links to all related documentation for this feature  
> **Audience**: Developers, AI Assistants

---

## Architecture Decision Records (ADRs)

| ADR | Title | Why Relevant |
|-----|-------|--------------|
| [ADR-001](../../docs/decisions/adr-001-microservices.md) | Microservices Architecture | Post Service is a separate microservice |
| [ADR-002](../../docs/decisions/adr-002-postgresql.md) | PostgreSQL Database | Posts stored in PostgreSQL |
| [ADR-004](../../docs/decisions/adr-004-rabbitmq.md) | RabbitMQ for Events | Post events (created, published) via RabbitMQ |
| ADR-007 (pending) | CDN Strategy | Image delivery via CloudFront (in review) |

**Read these ADRs to understand**:
- Why we chose microservices over monolith
- Why PostgreSQL for relational data
- How to publish events correctly
- CDN strategy for images (once approved)

---

## Components & Services

### Post Service (port 3002)

**Responsibilities**:
- Blog post CRUD operations
- Image upload to S3
- Full-text search
- Post analytics
- Event publishing

**Technology**:
- Node.js 18+ (Express.js)
- PostgreSQL (posts schema)
- AWS SDK (S3 integration)
- RabbitMQ client

**Configuration**:
```bash
# Required environment variables
PORT=3002
DB_HOST=...
DB_NAME=blogengine_prod
AWS_S3_BUCKET=blogengine-uploads
RABBITMQ_URL=...
```

See: [System Architecture - Post Service](../../docs/architecture/system-overview.md#post-service)

### Database: Posts Schema

**Tables**:
- `posts` - Main post data
- `post_tags` - Tag relationships
- `post_likes` - Like tracking

**Key indexes**:
- `idx_posts_author` on `author_id`
- `idx_posts_status` on `status`
- `idx_posts_created` on `created_at`
- `idx_posts_fulltext` on `tsvector`

See: [Database Schema](../../reference/database/schema.md#posts)

### AWS S3 Bucket

**Bucket**: `blogengine-uploads-prod`  
**Region**: `us-east-1`  
**Access**: Private (presigned URLs for upload/download)  
**Lifecycle**: Delete after 30 days if post deleted

See: [Configuration Parameters](../../reference/configuration/parameters.md#post-service-configuration)

### RabbitMQ Exchange

**Exchange**: `blog.events` (topic)  
**Routing keys**:
- `post.created`
- `post.published`
- `post.updated`
- `post.deleted`

**Consumers**:
- Notification Service (sends notifications to followers)
- Analytics Service (future - tracks post metrics)

See: [Event Schema](../../docs/protocols/event-schema.md)

---

## Security Requirements

### Authentication (CC6.1)

**Required for**:
- Create post: Valid JWT token
- Update post: Valid JWT + (owner or admin)
- Delete post: Valid JWT + (owner or admin)
- Like post: Valid JWT token

**Not required for**:
- List posts (public)
- Get single post (public)

See: [Authentication](../../docs/security/authentication.md)

### Authorization (CC6.1)

**Rules**:
1. Only post owner can update/delete their posts
2. Admin role can update/delete any post
3. Published posts visible to all
4. Draft posts visible only to owner/admin

**Implementation**:
```javascript
// Check authorization
if (post.authorId !== userId && userRole !== 'admin') {
  throw new ForbiddenError('Not authorized to modify this post');
}
```

See: [Authorization & RBAC](../../docs/security/authorization.md)

### Input Validation (CC7.2)

**Validate all inputs**:
- Title: 5-200 characters, no HTML tags
- Content: 10-50,000 characters, sanitize HTML
- Tags: Max 10 tags, alphanumeric only
- Images: JPEG/PNG/GIF only, max 10MB

**XSS Prevention**:
- Sanitize HTML content (DOMPurify)
- Set Content-Security-Policy header
- Escape user input in responses

See: [API Security](../../docs/security/api-security.md)

### Data Protection (CC6.6)

**At rest**:
- Database encrypted (AWS RDS encryption)
- S3 bucket encrypted (AES-256)

**In transit**:
- HTTPS only (TLS 1.3)
- Presigned S3 URLs expire after 1h

See: [Data Protection](../../docs/security/data-protection.md)

---

## API Documentation

### Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/posts` | No | List posts (paginated) |
| GET | `/posts/:id` | No | Get single post |
| POST | `/posts` | Yes | Create post |
| PUT | `/posts/:id` | Yes (owner/admin) | Update post |
| DELETE | `/posts/:id` | Yes (owner/admin) | Delete post |
| POST | `/posts/:id/like` | Yes | Like post |
| PUT | `/posts/:id/publish` | Yes (owner/admin) | Publish draft |

**Request/Response examples**: See [API Reference](../../docs/api/rest-api.md#posts)

---

## Protocols & Conventions

### REST API Conventions

**Naming**: Use plural nouns (`/posts` not `/post`)  
**Methods**: POST (create), GET (read), PUT (update), DELETE (delete)  
**Status codes**: 200 (OK), 201 (Created), 400 (Bad Request), 401 (Unauthorized), 403 (Forbidden), 404 (Not Found)  
**Pagination**: Query params `limit`, `offset`  
**Filtering**: Query params `status`, `author`, `tag`

See: [API Conventions](../../docs/protocols/api-conventions.md)

### Event Schema

**All events follow standard schema**:
```json
{
  "eventType": "post.created",
  "eventId": "uuid",
  "timestamp": "ISO 8601",
  "payload": { /* event-specific data */ }
}
```

See: [Event Schema](../../docs/protocols/event-schema.md)

### Error Handling

**Standard error response**:
```json
{
  "error": {
    "code": "INVALID_INPUT",
    "message": "Title must be between 5-200 characters",
    "field": "title",
    "requestId": "uuid"
  }
}
```

See: [Error Handling](../../docs/protocols/error-handling.md)

### Logging

**Log all operations**:
- Post created/updated/deleted (INFO level)
- Failed authorization attempts (WARN level)
- Errors (ERROR level)

**Include in logs**:
- Request ID (correlation)
- User ID (audit trail)
- Post ID
- Action taken

See: [Logging Standards](../../docs/protocols/logging-standards.md)

---

## Testing Documentation

**Test scenarios**: See [testing.md](testing.md) in this folder

**Test coverage target**: 80%  
**Current coverage**: 72%

**Tools**:
- Unit tests: Jest
- Integration tests: Supertest
- Load tests: k6

---

## Related Features

### Dependencies (Required)

| Feature | Dependency Type | Why |
|---------|----------------|-----|
| [User Authentication](../user-authentication/) | Hard dependency | Need auth to create/update posts |
| Database Schema | Hard dependency | Posts table must exist |

### Dependents (Features that need this)

| Feature | Dependency Type | Why |
|---------|----------------|-----|
| [Comment System](../comment-system/) | Hard dependency | Comments reference post_id |
| [Notifications](../notifications/) | Soft dependency | Notify followers on new post |
| [Admin Panel](../admin-panel/) | Soft dependency | Moderate posts |

---

## External Resources

**Libraries used**:
- Express.js - Web framework
- Sequelize - PostgreSQL ORM
- AWS SDK - S3 integration
- DOMPurify - HTML sanitization
- Sharp - Image processing

**Relevant reading**:
- [REST API Best Practices](https://restfulapi.net/)
- [PostgreSQL Full-Text Search](https://www.postgresql.org/docs/current/textsearch.html)
- [AWS S3 Presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/PresignedUrlUploadObject.html)

---

## For AI Assistants

**To implement a task in this feature**, load these files:

```bash
# Feature context
cat features/blog-posts/*.md

# Architectural context
cat docs/decisions/adr-001-microservices.md
cat docs/decisions/adr-002-postgresql.md
cat docs/decisions/adr-004-rabbitmq.md

# Security context
cat docs/security/authentication.md
cat docs/security/authorization.md
cat docs/security/api-security.md

# Protocol context
cat docs/protocols/api-conventions.md
cat docs/protocols/event-schema.md
cat docs/protocols/error-handling.md
```

**This provides**:
- What to build (requirements, design, tasks)
- How to build it (ADRs, protocols)
- How to secure it (security docs)
- How to integrate it (API, events)

---

**Last Updated**: November 5, 2024  
**Maintained By**: Backend Team B
