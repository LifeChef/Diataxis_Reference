# Feature: Blog Posts

> **Status**: 🔄 In Progress (60% complete)  
> **Sprint**: 12  
> **Owner**: Backend Team B  
> **Priority**: P0 (MVP Critical)

---

## Overview

Core blog posting functionality: create, read, update, delete blog posts with rich text content, image uploads, tags, and draft/publish workflow.

**What's working**:
- ✅ Create post (draft/published)
- ✅ List posts (with pagination)
- ✅ Get single post
- ✅ Update post metadata
- ✅ Tag system

**In progress**:
- 🔄 Image upload to S3/CDN (blocked: performance issues)
- 🔄 Full-text search
- 🔄 Post likes/reactions

**Not started**:
- 📋 Delete post (soft delete)
- 📋 Post scheduling
- 📋 Post analytics (views, engagement)

---

## Quick Links

- **Requirements**: [requirements.md](requirements.md) - User stories, acceptance criteria
- **Design**: [design.md](design.md) - Technical architecture, data model
- **Tasks**: [tasks.md](tasks.md) - Implementation checklist (12/20 done)
- **Testing**: [testing.md](testing.md) - Test scenarios
- **References**: [references.md](references.md) - ADRs, components, security

---

## Sprint Goals

### Sprint 12 (Current)
- [x] Basic CRUD operations
- [ ] Image upload with CDN (blocked - investigating CloudFront)
- [ ] Full-text search
- [x] Tag system

### Sprint 13 (Next)
- [ ] Post likes/reactions
- [ ] Post scheduling
- [ ] Soft delete
- [ ] Analytics foundation

---

## Key Metrics

| Metric | Target | Current | Status |
|--------|--------|---------|--------|
| API response time (list posts) | <100ms | 85ms | ✅ Good |
| API response time (create post) | <200ms | 150ms | ✅ Good |
| Image upload time (5MB) | <3s | ~8s | ⚠️ **Blocked** |
| Test coverage | 80% | 72% | 🔄 Improving |

---

## Blockers

### 🔴 Critical: Image Upload Performance
**Issue**: Large images (>5MB) causing timeout  
**Impact**: Cannot complete Sprint 12 goal  
**Owner**: DevOps + Backend  
**Action**: Evaluating CloudFront CDN + S3 direct upload  
**Target Resolution**: Sprint 12 (this week)  
**Related**: ADR-007 (pending)

---

## API Endpoints

Implemented by Post Service (port 3002):

- `GET /posts` - List posts (paginated)
- `GET /posts/:id` - Get single post
- `POST /posts` - Create post (auth required)
- `PUT /posts/:id` - Update post (owner/admin only)
- `DELETE /posts/:id` - Delete post (not implemented)
- `POST /posts/:id/like` - Like post (not implemented)

See [API Reference](../../docs/api/rest-api.md#posts)

---

## Database Tables

- `posts` - Main post data
- `post_tags` - Post-tag relationships
- `post_likes` - Post likes (future)

See [Database Schema](../../reference/database/schema.md#posts)

---

## Team

**Owner**: Sarah Johnson (Backend Team B)  
**Contributors**:
- John Doe (Backend) - API implementation
- Jane Smith (Backend) - Image upload
- Bob Wilson (DevOps) - CDN setup

**Stakeholder**: Product Manager (Alex Lee)

---

## Related Features

**Dependencies**:
- [User Authentication](../user-authentication/) - Auth required for create/update
- [Comment System](../comment-system/) - Comments reference posts

**Future dependencies**:
- [Admin Panel](../admin-panel/) - Content moderation
- [Notifications](../notifications/) - Notify followers on new post

---

**Last Updated**: November 5, 2024  
**Next Review**: Sprint 13 Planning (November 11, 2024)
