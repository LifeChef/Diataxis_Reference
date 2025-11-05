# Blog Posts - Implementation Tasks

**Status**: In Progress (12/20 complete - 60%)  
**Owner**: Backend Team B  
**Sprint**: Sprint 12

---

## Task Checklist

### API Endpoints (6/8 complete)

- [x] **Create post endpoint** `POST /posts`
  - Owner: John
  - Components: Post Service, PostgreSQL
  - Security: JWT auth required, owner validation
  - Completed: Sprint 11

- [x] **List posts endpoint** `GET /posts`
  - Owner: John
  - Features: Pagination (limit, offset), filtering (status, author, tag)
  - Performance: < 100ms response time
  - Completed: Sprint 11

- [x] **Get single post** `GET /posts/:id`
  - Owner: John
  - Includes: Post data, author info, tags, like count
  - Completed: Sprint 11

- [x] **Update post endpoint** `PUT /posts/:id`
  - Owner: John
  - Security: Owner or admin only
  - Validation: All fields optional
  - Completed: Sprint 12

- [ ] **Delete post endpoint** `DELETE /posts/:id`
  - Owner: John
  - Implementation: Soft delete (set deleted_at)
  - Security: Owner or admin only
  - Status: Not started
  - Sprint: 13

- [ ] **Like post endpoint** `POST /posts/:id/like`
  - Owner: Sarah
  - Implementation: Upsert in post_likes table
  - Auth: Required
  - Status: Not started
  - Sprint: 13

- [x] **Publish post** `PUT /posts/:id/publish`
  - Owner: John
  - Changes status from draft to published
  - Event: Triggers post.published event
  - Completed: Sprint 12

- [ ] **Schedule post** `PUT /posts/:id/schedule`
  - Owner: TBD
  - Implementation: Cron job checks scheduled posts
  - Status: Not started
  - Sprint: 13

---

### Data Model (4/4 complete)

- [x] **Posts table schema**
  - Columns: id, title, content, author_id, status, created_at, updated_at, deleted_at
  - Indexes: author_id, status, created_at
  - Completed: Sprint 11
  - See: [Database Schema](../../reference/database/schema.md#posts)

- [x] **Post tags relationship**
  - Table: post_tags (post_id, tag_id)
  - Many-to-many relationship
  - Completed: Sprint 11

- [x] **Post likes table** (structure only)
  - Table: post_likes (id, post_id, user_id, created_at)
  - Unique constraint on (post_id, user_id)
  - Completed: Sprint 12

- [x] **Database migrations**
  - Tool: Flyway (ADR-006)
  - All migrations versioned
  - Completed: Sprint 11

---

### Image Upload (0/4 complete) - 🔴 BLOCKED

- [ ] **S3 upload integration**
  - Owner: Jane
  - Implementation: AWS SDK, presigned URLs
  - Security: Validate file type, size limit (10MB)
  - Status: In progress
  - Blocker: Performance issues with large files (>5MB)
  - Sprint: 12

- [ ] **CDN integration (CloudFront)**
  - Owner: Bob (DevOps)
  - Setup: CloudFront distribution for S3 bucket
  - Status: Investigating
  - Related: ADR-007 (pending)
  - Sprint: 12

- [ ] **Image optimization**
  - Owner: Jane
  - Implementation: Resize on upload (thumbnail, medium, large)
  - Library: Sharp.js
  - Status: Not started (depends on S3 upload)
  - Sprint: 13

- [ ] **Image deletion cleanup**
  - Owner: Jane
  - Implementation: Delete from S3 when post deleted
  - Status: Not started
  - Sprint: 13

---

### Full-Text Search (1/3 complete)

- [x] **PostgreSQL full-text search setup**
  - Column: tsvector for title + content
  - Index: GIN index on tsvector
  - Completed: Sprint 11

- [ ] **Search endpoint** `GET /posts/search?q=...`
  - Owner: Sarah
  - Implementation: ts_query with ranking
  - Status: In progress
  - Sprint: 12

- [ ] **Search relevance tuning**
  - Owner: Sarah
  - Implementation: Weight title higher than content
  - Status: Not started
  - Sprint: 13

---

### Event Publishing (1/1 complete)

- [x] **RabbitMQ integration**
  - Events: post.created, post.published, post.updated, post.deleted
  - Owner: John
  - Completed: Sprint 12
  - See: [Event Schema](../../docs/protocols/event-schema.md)

**Events published**:
```javascript
// post.created
{
  "eventType": "post.created",
  "eventId": "uuid",
  "timestamp": "November 5, 2024T10:00:00Z",
  "payload": {
    "postId": "uuid",
    "authorId": "uuid",
    "title": "Post title",
    "status": "draft"
  }
}

// post.published
{
  "eventType": "post.published",
  "eventId": "uuid",
  "timestamp": "November 5, 2024T10:05:00Z",
  "payload": {
    "postId": "uuid",
    "authorId": "uuid",
    "title": "Post title"
  }
}
```

---

### Testing (0/5 complete)

- [ ] **Unit tests**
  - Target: 80% coverage
  - Current: 72%
  - Owner: Team
  - Sprint: 12-13

- [ ] **Integration tests**
  - Scenarios: Create → Read → Update → Delete
  - Owner: QA (Maria)
  - Sprint: 13

- [ ] **Performance tests**
  - Target: List 1000 posts < 100ms
  - Tool: k6
  - Owner: DevOps (Bob)
  - Sprint: 13

- [ ] **Security tests**
  - Authorization: Non-owner cannot update post
  - Input validation: XSS prevention
  - Owner: Security Team
  - Sprint: 13

- [ ] **Load tests**
  - Target: 1000 concurrent users
  - Owner: DevOps (Bob)
  - Sprint: 17 (performance phase)

---

## Task Dependencies

```
Database schema → API endpoints → Image upload
                              → Full-text search
                              → Event publishing

API endpoints → Testing
Image upload → Testing
```

---

## References

**Components**:
- Post Service (port 3002)
- PostgreSQL (posts schema)
- RabbitMQ (events)
- AWS S3 (image storage)
- CloudFront (CDN - pending)

**ADRs**:
- [ADR-001: Microservices](../../docs/decisions/adr-001-microservices.md)
- [ADR-002: PostgreSQL](../../docs/decisions/adr-002-postgresql.md)
- [ADR-004: RabbitMQ](../../docs/decisions/adr-004-rabbitmq.md)
- ADR-007: CDN Strategy (pending)

**Security**:
- [Authorization](../../docs/security/authorization.md) - Post ownership validation
- [API Security](../../docs/security/api-security.md) - Input validation
- [Data Protection](../../docs/security/data-protection.md) - Image upload security

**APIs**:
- [REST API: Posts](../../docs/api/rest-api.md#posts)

---

## Notes

**Performance targets**:
- List posts: <100ms (P95)
- Create post: <200ms (P95)
- Image upload: <3s for 5MB (not met yet)

**Next sprint priorities**:
1. Resolve image upload blocker
2. Complete full-text search
3. Implement post likes
4. Increase test coverage to 80%

---

**Last Updated**: November 5, 2024  
**Sprint**: 12 of 16
