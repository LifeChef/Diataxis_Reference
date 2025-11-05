# Blog Posts - Technical Design

> **Feature**: Blog Posts  
> **Architect**: Sarah Johnson  
> **Last Updated**: November 5, 2024  
> **Status**: ✅ Design Approved

---

## Overview

This document outlines the technical design for the Blog Posts feature, including architecture, data models, API contracts, and implementation details.

**Design Principles**:
- **Scalability**: Support 10K+ posts and 1,000 concurrent readers
- **Performance**: < 1 second page loads, efficient caching
- **Security**: Input sanitization, authorization checks
- **Maintainability**: Clean separation of concerns, well-tested

---

## Architecture

### Service Architecture

```
┌─────────────┐
│   Client    │ (Web/Mobile)
│ (React/RN)  │
└──────┬──────┘
       │
       │ HTTPS
       │
┌──────▼────────┐
│  API Gateway  │ :3000
│               │ - Auth validation
│               │ - Rate limiting
└──────┬────────┘
       │
       │ REST
       │
┌──────▼────────┐
│ Post Service  │ :3002
│               │
│ ┌───────────┐ │
│ │Controller │ │
│ └─────┬─────┘ │
│       │       │
│ ┌─────▼─────┐ │
│ │  Service  │ │ - Business logic
│ └─────┬─────┘ │ - Validation
│       │       │
│ ┌─────▼─────┐ │
│ │Repository │ │ - Data access
│ └─────┬─────┘ │
└───────┼───────┘
        │
    ┌───▼────┐     ┌─────────┐     ┌──────────┐
    │PostGres│     │  Redis  │     │   S3     │
    │   DB   │     │  Cache  │     │ (Images) │
    └────────┘     └─────────┘     └──────────┘
```

### Component Layers

**1. Controller Layer** (`controllers/post.controller.js`)
- Handle HTTP requests/responses
- Input validation (express-validator)
- Call service layer methods
- Error handling

**2. Service Layer** (`services/post.service.js`)
- Business logic implementation
- Transaction management
- Event publishing (RabbitMQ)
- Cache management

**3. Repository Layer** (`repositories/post.repository.js`)
- Database queries (Knex.js ORM)
- Query optimization
- Connection pooling

**4. Model Layer** (`models/post.model.js`)
- Data validation schemas
- Type definitions
- Constants

---

## Data Models

### Database Schema

#### Posts Table

```sql
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  author_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  title VARCHAR(200) NOT NULL,
  slug VARCHAR(250) UNIQUE NOT NULL,
  content TEXT NOT NULL,
  excerpt VARCHAR(300),
  status VARCHAR(20) NOT NULL DEFAULT 'draft',
    -- Values: 'draft', 'published', 'archived', 'scheduled'
  featured_image_url VARCHAR(500),
  view_count INTEGER DEFAULT 0,
  like_count INTEGER DEFAULT 0,
  comment_count INTEGER DEFAULT 0,
  published_at TIMESTAMP,
  scheduled_for TIMESTAMP,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
  
  -- Indexes
  INDEX idx_author_id (author_id),
  INDEX idx_status (status),
  INDEX idx_published_at (published_at DESC),
  INDEX idx_slug (slug),
  INDEX idx_status_published (status, published_at DESC) 
    WHERE status = 'published'
);

-- Trigger for auto-updating updated_at
CREATE TRIGGER update_posts_updated_at
  BEFORE UPDATE ON posts
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();
```

#### Post Tags Table (Many-to-Many)

```sql
CREATE TABLE tags (
  id SERIAL PRIMARY KEY,
  name VARCHAR(50) UNIQUE NOT NULL,
  slug VARCHAR(50) UNIQUE NOT NULL,
  post_count INTEGER DEFAULT 0,
  created_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE TABLE post_tags (
  post_id INTEGER NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
  tag_id INTEGER NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  
  PRIMARY KEY (post_id, tag_id),
  INDEX idx_tag_id (tag_id),
  INDEX idx_post_id (post_id)
);

-- Trigger to update tag.post_count
CREATE TRIGGER update_tag_post_count_on_insert
  AFTER INSERT ON post_tags
  FOR EACH ROW
  EXECUTE FUNCTION increment_tag_post_count();

CREATE TRIGGER update_tag_post_count_on_delete
  AFTER DELETE ON post_tags
  FOR EACH ROW
  EXECUTE FUNCTION decrement_tag_post_count();
```

#### Post Likes Table

```sql
CREATE TABLE post_likes (
  id SERIAL PRIMARY KEY,
  post_id INTEGER NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
  user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  
  UNIQUE(post_id, user_id),
  INDEX idx_post_id (post_id),
  INDEX idx_user_id (user_id)
);

-- Trigger to update posts.like_count
CREATE TRIGGER update_post_like_count_on_insert
  AFTER INSERT ON post_likes
  FOR EACH ROW
  EXECUTE FUNCTION increment_post_like_count();

CREATE TRIGGER update_post_like_count_on_delete
  AFTER DELETE ON post_likes
  FOR EACH ROW
  EXECUTE FUNCTION decrement_post_like_count();
```

#### Post Views Table (for analytics)

```sql
CREATE TABLE post_views (
  id SERIAL PRIMARY KEY,
  post_id INTEGER NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
  user_id INTEGER REFERENCES users(id) ON DELETE SET NULL, -- NULL for anonymous
  ip_address INET,
  user_agent VARCHAR(500),
  referrer VARCHAR(500),
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  
  INDEX idx_post_id (post_id),
  INDEX idx_created_at (created_at DESC),
  INDEX idx_post_created (post_id, created_at DESC)
);

-- Partition by month for better performance
CREATE TABLE post_views_2024_11 PARTITION OF post_views
  FOR VALUES FROM ('2024-11-01') TO ('2024-12-01');
```

---

### TypeScript Interfaces

```typescript
// models/post.model.ts

export interface Post {
  id: number;
  authorId: number;
  title: string;
  slug: string;
  content: string;
  excerpt?: string;
  status: PostStatus;
  featuredImageUrl?: string;
  viewCount: number;
  likeCount: number;
  commentCount: number;
  publishedAt?: Date;
  scheduledFor?: Date;
  createdAt: Date;
  updatedAt: Date;
}

export enum PostStatus {
  DRAFT = 'draft',
  PUBLISHED = 'published',
  ARCHIVED = 'archived',
  SCHEDULED = 'scheduled'
}

export interface CreatePostDTO {
  title: string;
  content: string;
  excerpt?: string;
  tags?: string[];
  featuredImage?: File;
  status?: PostStatus;
  scheduledFor?: Date;
}

export interface UpdatePostDTO {
  title?: string;
  content?: string;
  excerpt?: string;
  tags?: string[];
  featuredImage?: File;
  status?: PostStatus;
}

export interface PostListResponse {
  posts: PostSummary[];
  pagination: {
    page: number;
    perPage: number;
    total: number;
    totalPages: number;
  };
}

export interface PostSummary {
  id: number;
  title: string;
  slug: string;
  excerpt: string;
  author: {
    id: number;
    name: string;
    avatarUrl: string;
  };
  tags: Tag[];
  viewCount: number;
  likeCount: number;
  commentCount: number;
  publishedAt: Date;
}

export interface PostDetail extends PostSummary {
  content: string;
  isLikedByCurrentUser: boolean;
  relatedPosts: PostSummary[];
}
```

---

## API Design

### REST Endpoints

#### List Posts
```http
GET /api/posts?page=1&limit=20&status=published&tag=technology&author=123&sort=recent

Response: 200 OK
{
  "posts": [
    {
      "id": 1,
      "title": "Getting Started with Node.js",
      "slug": "getting-started-with-nodejs",
      "excerpt": "Learn the basics of Node.js development...",
      "author": {
        "id": 123,
        "name": "John Doe",
        "avatarUrl": "https://cdn.example.com/avatars/john.jpg"
      },
      "tags": [
        { "id": 1, "name": "Node.js", "slug": "nodejs" },
        { "id": 2, "name": "JavaScript", "slug": "javascript" }
      ],
      "featuredImageUrl": "https://cdn.example.com/images/nodejs.jpg",
      "viewCount": 1250,
      "likeCount": 45,
      "commentCount": 12,
      "publishedAt": "2024-11-01T10:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "perPage": 20,
    "total": 156,
    "totalPages": 8
  }
}
```

#### Get Single Post
```http
GET /api/posts/:id or /api/posts/slug/:slug

Response: 200 OK
{
  "id": 1,
  "title": "Getting Started with Node.js",
  "slug": "getting-started-with-nodejs",
  "content": "# Introduction\n\nNode.js is a JavaScript runtime...",
  "excerpt": "Learn the basics of Node.js development...",
  "author": {
    "id": 123,
    "name": "John Doe",
    "avatarUrl": "https://cdn.example.com/avatars/john.jpg"
  },
  "tags": [...],
  "featuredImageUrl": "https://cdn.example.com/images/nodejs.jpg",
  "viewCount": 1251,
  "likeCount": 45,
  "commentCount": 12,
  "publishedAt": "2024-11-01T10:00:00Z",
  "isLikedByCurrentUser": false,
  "relatedPosts": [...]
}
```

#### Create Post
```http
POST /api/posts
Authorization: Bearer <token>
Content-Type: application/json

{
  "title": "My New Blog Post",
  "content": "This is the content...",
  "excerpt": "A brief summary",
  "tags": ["nodejs", "javascript"],
  "status": "draft"
}

Response: 201 Created
{
  "id": 157,
  "title": "My New Blog Post",
  "slug": "my-new-blog-post",
  "status": "draft",
  "createdAt": "2024-11-05T14:30:00Z"
}
```

#### Update Post
```http
PUT /api/posts/:id
Authorization: Bearer <token>
Content-Type: application/json

{
  "title": "Updated Title",
  "status": "published"
}

Response: 200 OK
{
  "id": 157,
  "title": "Updated Title",
  "slug": "updated-title",
  "status": "published",
  "publishedAt": "2024-11-05T14:35:00Z"
}
```

#### Delete/Archive Post
```http
DELETE /api/posts/:id
Authorization: Bearer <token>

Response: 204 No Content
```

#### Like Post
```http
POST /api/posts/:id/like
Authorization: Bearer <token>

Response: 201 Created
{
  "postId": 1,
  "likeCount": 46,
  "isLiked": true
}
```

#### Unlike Post
```http
DELETE /api/posts/:id/like
Authorization: Bearer <token>

Response: 200 OK
{
  "postId": 1,
  "likeCount": 45,
  "isLiked": false
}
```

---

## Business Logic

### Post Creation Flow

```javascript
// services/post.service.js

async createPost(authorId, createPostDTO) {
  // 1. Validate input
  const errors = validateCreatePost(createPostDTO);
  if (errors.length > 0) {
    throw new ValidationError(errors);
  }

  // 2. Generate slug
  const slug = await this.generateUniqueSlug(createPostDTO.title);

  // 3. Generate excerpt if not provided
  const excerpt = createPostDTO.excerpt || 
    generateExcerpt(createPostDTO.content, 150);

  // 4. Create post in database
  const post = await this.postRepository.create({
    authorId,
    title: createPostDTO.title,
    slug,
    content: createPostDTO.content,
    excerpt,
    status: createPostDTO.status || 'draft',
    scheduledFor: createPostDTO.scheduledFor
  });

  // 5. Handle tags
  if (createPostDTO.tags && createPostDTO.tags.length > 0) {
    await this.tagService.attachTagsToPost(post.id, createPostDTO.tags);
  }

  // 6. Handle featured image upload
  if (createPostDTO.featuredImage) {
    const imageUrl = await this.uploadService.uploadImage(
      createPostDTO.featuredImage,
      `posts/${post.id}`
    );
    await this.postRepository.update(post.id, { featuredImageUrl: imageUrl });
    post.featuredImageUrl = imageUrl;
  }

  // 7. Publish event
  if (post.status === 'published') {
    await this.eventPublisher.publish('post.created', {
      postId: post.id,
      authorId: post.authorId,
      title: post.title
    });
  }

  // 8. Clear relevant caches
  await this.cacheService.invalidate(`posts:author:${authorId}`);
  
  return post;
}
```

### Slug Generation

```javascript
async generateUniqueSlug(title) {
  let slug = slugify(title, { lower: true, strict: true });
  
  // Check if slug exists
  let counter = 1;
  while (await this.postRepository.slugExists(slug)) {
    slug = `${slugify(title)}-${counter}`;
    counter++;
  }
  
  return slug;
}
```

### View Tracking

```javascript
async trackView(postId, userId, ipAddress, userAgent, referrer) {
  // Deduplicate views: max 1 view per user per post per day
  const today = startOfDay(new Date());
  const existingView = await this.viewRepository.findOne({
    postId,
    userId: userId || null,
    createdAt: { gte: today }
  });
  
  if (existingView) {
    return; // Already counted today
  }
  
  // Record view
  await this.viewRepository.create({
    postId,
    userId,
    ipAddress,
    userAgent,
    referrer
  });
  
  // Increment view count (async, don't block response)
  this.postRepository.incrementViewCount(postId).catch(err => {
    logger.error('Failed to increment view count', { postId, error: err });
  });
}
```

---

## Caching Strategy

### Cache Layers

**1. CDN Caching** (CloudFront)
- **What**: Published post HTML pages
- **TTL**: 5 minutes
- **Invalidation**: On post update/delete

**2. Redis Caching**
- **What**: Post list queries, post details, tag lists
- **TTL**: Varies by type

```javascript
// Cache keys and TTLs
const CACHE_CONFIG = {
  'post:detail:{id}': 3600,        // 1 hour
  'post:slug:{slug}': 3600,         // 1 hour
  'posts:list:{page}:{filters}': 300,  // 5 minutes
  'posts:author:{authorId}': 600,   // 10 minutes
  'tags:popular': 3600,             // 1 hour
  'post:related:{id}': 1800         // 30 minutes
};
```

### Cache Invalidation

```javascript
async invalidatePostCaches(postId, authorId) {
  await Promise.all([
    this.cache.del(`post:detail:${postId}`),
    this.cache.del(`posts:author:${authorId}`),
    this.cache.delPattern('posts:list:*'),  // Invalidate all list caches
    this.cache.del(`post:related:*`)  // Related posts might change
  ]);
}
```

---

## Image Upload Flow

```javascript
async uploadImage(file, path) {
  // 1. Validate file
  if (!ALLOWED_IMAGE_TYPES.includes(file.mimetype)) {
    throw new ValidationError('Invalid image type');
  }
  
  if (file.size > MAX_IMAGE_SIZE) {
    throw new ValidationError('Image too large (max 5MB)');
  }

  // 2. Generate unique filename
  const filename = `${uuidv4()}${extname(file.originalname)}`;
  const key = `${path}/${filename}`;

  // 3. Optimize image
  const optimized = await sharp(file.buffer)
    .resize(1200, 800, { fit: 'inside', withoutEnlargement: true })
    .jpeg({ quality: 85 })
    .toBuffer();

  // 4. Upload to S3
  await this.s3.putObject({
    Bucket: process.env.S3_BUCKET,
    Key: key,
    Body: optimized,
    ContentType: file.mimetype,
    CacheControl: 'max-age=31536000',  // 1 year
    ACL: 'public-read'
  });

  // 5. Return CDN URL
  return `${process.env.CDN_URL}/${key}`;
}
```

---

## Search Implementation

### Elasticsearch Integration

```javascript
// Index post on creation/update
async indexPost(post) {
  await this.elasticsearchClient.index({
    index: 'posts',
    id: post.id,
    body: {
      title: post.title,
      content: post.content,
      excerpt: post.excerpt,
      author: post.author.name,
      tags: post.tags.map(t => t.name),
      status: post.status,
      publishedAt: post.publishedAt
    }
  });
}

// Search posts
async searchPosts(query, { page = 1, limit = 20 }) {
  const result = await this.elasticsearchClient.search({
    index: 'posts',
    body: {
      query: {
        bool: {
          must: [
            {
              multi_match: {
                query: query,
                fields: ['title^3', 'content', 'tags^2', 'author'],
                type: 'best_fields',
                fuzziness: 'AUTO'
              }
            },
            {
              term: { status: 'published' }
            }
          ]
        }
      },
      highlight: {
        fields: {
          title: {},
          content: { fragment_size: 150 }
        }
      },
      from: (page - 1) * limit,
      size: limit,
      sort: ['_score', { publishedAt: 'desc' }]
    }
  });

  return this.formatSearchResults(result);
}
```

---

## Performance Optimizations

### Database Query Optimization

```sql
-- Use covering index for post list query
CREATE INDEX idx_posts_list_covering ON posts (
  status, published_at DESC
) INCLUDE (id, author_id, title, slug, excerpt, view_count, like_count, comment_count);

-- Partial index for published posts only
CREATE INDEX idx_published_posts ON posts (published_at DESC)
WHERE status = 'published';
```

### N+1 Query Prevention

```javascript
// BAD: N+1 query problem
async getPostsWithAuthors() {
  const posts = await db('posts').select('*');
  for (const post of posts) {
    post.author = await db('users').where({ id: post.author_id }).first();
  }
  return posts;
}

// GOOD: Single query with JOIN
async getPostsWithAuthors() {
  return await db('posts')
    .join('users', 'posts.author_id', 'users.id')
    .select(
      'posts.*',
      'users.id as author_id',
      'users.name as author_name',
      'users.avatar_url as author_avatar'
    )
    .where('posts.status', 'published')
    .orderBy('posts.published_at', 'desc');
}
```

---

## Security Considerations

### Authorization Checks

```javascript
// Middleware to check post ownership
async checkPostOwnership(req, res, next) {
  const post = await postService.getById(req.params.id);
  
  if (!post) {
    return res.status(404).json({ error: 'Post not found' });
  }
  
  // Allow post author or admin
  if (post.authorId !== req.user.id && req.user.role !== 'admin') {
    return res.status(403).json({ error: 'Forbidden' });
  }
  
  req.post = post;
  next();
}
```

### Input Sanitization

```javascript
import DOMPurify from 'isomorphic-dompurify';

function sanitizePostContent(content) {
  // Allow safe HTML but remove scripts, etc.
  return DOMPurify.sanitize(content, {
    ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'u', 'h1', 'h2', 'h3', 
                   'ul', 'ol', 'li', 'a', 'img', 'code', 'pre', 'blockquote'],
    ALLOWED_ATTR: ['href', 'src', 'alt', 'title', 'class']
  });
}
```

---

## Error Handling

```javascript
// Custom error classes
class PostNotFoundError extends Error {
  constructor(postId) {
    super(`Post with ID ${postId} not found`);
    this.name = 'PostNotFoundError';
    this.statusCode = 404;
  }
}

class PostValidationError extends Error {
  constructor(errors) {
    super('Post validation failed');
    this.name = 'PostValidationError';
    this.statusCode = 400;
    this.errors = errors;
  }
}

// Error handler middleware
function handlePostErrors(err, req, res, next) {
  if (err instanceof PostNotFoundError) {
    return res.status(404).json({
      error: {
        code: 'POST_NOT_FOUND',
        message: err.message
      }
    });
  }
  
  if (err instanceof PostValidationError) {
    return res.status(400).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: err.message,
        details: err.errors
      }
    });
  }
  
  next(err);
}
```

---

## Testing Strategy

See [testing.md](testing.md) for comprehensive test coverage.

---

## Related Documentation

- [Requirements](requirements.md)
- [Tasks Checklist](tasks.md)
- [Testing Strategy](testing.md)
- [ADR-002: PostgreSQL](../../docs/decisions/adr-002-postgresql.md)
- [API Reference](../../reference/api/endpoints.md)

---

**Reviewed By**: Architecture Team, Security Team  
**Approval Date**: October 30, 2024  
**Status**: ✅ Approved for Implementation
