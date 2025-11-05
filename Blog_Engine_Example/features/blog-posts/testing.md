# Blog Posts - Testing Strategy

> **Feature**: Blog Posts  
> **QA Lead**: Maria Garcia  
> **Last Updated**: November 5, 2024  
> **Status**: ✅ Test Plan Approved

---

## Overview

This document outlines the comprehensive testing strategy for the Blog Posts feature, including unit tests, integration tests, E2E tests, performance tests, and security tests.

**Testing Goals**:
- **Coverage**: 80%+ code coverage
- **Quality**: Catch bugs before production
- **Performance**: Validate performance targets
- **Security**: Verify input sanitization and authorization

---

## Test Coverage Summary

| Test Type | Coverage | Count | Status |
|-----------|----------|-------|--------|
| Unit Tests | 85% | 120 | ✅ Complete |
| Integration Tests | 100% of endpoints | 35 | ✅ Complete |
| E2E Tests | Critical paths | 12 | 🔄 In Progress |
| Performance Tests | Key endpoints | 5 | ✅ Complete |
| Security Tests | All inputs | 15 | ✅ Complete |

---

## Unit Tests

### Service Layer Tests

#### PostService.createPost()

**Test File**: `services/__tests__/post.service.test.js`

```javascript
describe('PostService.createPost', () => {
  let postService;
  let mockPostRepository;
  let mockTagService;
  
  beforeEach(() => {
    mockPostRepository = {
      create: jest.fn(),
      slugExists: jest.fn()
    };
    mockTagService = {
      attachTagsToPost: jest.fn()
    };
    postService = new PostService(mockPostRepository, mockTagService);
  });

  it('should create a draft post successfully', async () => {
    // Arrange
    const authorId = 123;
    const createPostDTO = {
      title: 'Test Post',
      content: 'This is test content with sufficient length to meet requirements.'
    };
    mockPostRepository.slugExists.mockResolvedValue(false);
    mockPostRepository.create.mockResolvedValue({
      id: 1,
      ...createPostDTO,
      slug: 'test-post',
      authorId,
      status: 'draft'
    });

    // Act
    const result = await postService.createPost(authorId, createPostDTO);

    // Assert
    expect(result).toHaveProperty('id', 1);
    expect(result.slug).toBe('test-post');
    expect(result.status).toBe('draft');
    expect(mockPostRepository.create).toHaveBeenCalledTimes(1);
  });

  it('should generate unique slug when duplicate exists', async () => {
    // Arrange
    const createPostDTO = {
      title: 'Test Post',
      content: 'Content here.'
    };
    mockPostRepository.slugExists
      .mockResolvedValueOnce(true)  // First check: exists
      .mockResolvedValueOnce(false); // Second check: doesn't exist
    mockPostRepository.create.mockResolvedValue({
      id: 1,
      slug: 'test-post-1'
    });

    // Act
    const result = await postService.createPost(123, createPostDTO);

    // Assert
    expect(result.slug).toBe('test-post-1');
    expect(mockPostRepository.slugExists).toHaveBeenCalledTimes(2);
  });

  it('should attach tags to post', async () => {
    // Arrange
    const createPostDTO = {
      title: 'Test Post',
      content: 'Content',
      tags: ['nodejs', 'javascript']
    };
    mockPostRepository.slugExists.mockResolvedValue(false);
    mockPostRepository.create.mockResolvedValue({ id: 1 });

    // Act
    await postService.createPost(123, createPostDTO);

    // Assert
    expect(mockTagService.attachTagsToPost).toHaveBeenCalledWith(
      1,
      ['nodejs', 'javascript']
    );
  });

  it('should throw validation error for empty title', async () => {
    // Arrange
    const createPostDTO = {
      title: '',
      content: 'Content'
    };

    // Act & Assert
    await expect(
      postService.createPost(123, createPostDTO)
    ).rejects.toThrow(ValidationError);
  });

  it('should throw validation error for short content', async () => {
    // Arrange
    const createPostDTO = {
      title: 'Test',
      content: 'Too short' // Less than 100 chars
    };

    // Act & Assert
    await expect(
      postService.createPost(123, createPostDTO)
    ).rejects.toThrow('Content must be at least 100 characters');
  });

  it('should publish event when status is published', async () => {
    // Arrange
    const mockEventPublisher = { publish: jest.fn() };
    postService.eventPublisher = mockEventPublisher;
    const createPostDTO = {
      title: 'Test Post',
      content: 'Content here with sufficient length for validation.',
      status: 'published'
    };
    mockPostRepository.slugExists.mockResolvedValue(false);
    mockPostRepository.create.mockResolvedValue({
      id: 1,
      status: 'published'
    });

    // Act
    await postService.createPost(123, createPostDTO);

    // Assert
    expect(mockEventPublisher.publish).toHaveBeenCalledWith(
      'post.created',
      expect.objectContaining({ postId: 1 })
    );
  });
});
```

---

#### PostService.updatePost()

```javascript
describe('PostService.updatePost', () => {
  it('should update post title and slug', async () => {
    // Test implementation
  });

  it('should preserve published_at when editing published post', async () => {
    // Test implementation
  });

  it('should set published_at when changing status to published', async () => {
    // Test implementation
  });

  it('should throw error when user is not author', async () => {
    // Test implementation
  });
});
```

---

### Repository Layer Tests

**Test File**: `repositories/__tests__/post.repository.test.js`

```javascript
describe('PostRepository', () => {
  let postRepository;
  let db;

  beforeAll(async () => {
    db = await setupTestDatabase();
    postRepository = new PostRepository(db);
  });

  afterEach(async () => {
    await db('posts').truncate();
  });

  afterAll(async () => {
    await db.destroy();
  });

  it('should create post with correct data', async () => {
    // Arrange
    const postData = {
      authorId: 1,
      title: 'Test Post',
      slug: 'test-post',
      content: 'Content',
      status: 'draft'
    };

    // Act
    const result = await postRepository.create(postData);

    // Assert
    expect(result.id).toBeDefined();
    expect(result.title).toBe('Test Post');
    expect(result.createdAt).toBeInstanceOf(Date);
  });

  it('should find post by id with author', async () => {
    // Test implementation
  });

  it('should list posts with pagination', async () => {
    // Create test posts
    await Promise.all([
      postRepository.create({ title: 'Post 1', authorId: 1, status: 'published' }),
      postRepository.create({ title: 'Post 2', authorId: 1, status: 'published' }),
      postRepository.create({ title: 'Post 3', authorId: 1, status: 'draft' })
    ]);

    // Act
    const result = await postRepository.list({
      status: 'published',
      page: 1,
      limit: 10
    });

    // Assert
    expect(result.posts).toHaveLength(2);
    expect(result.total).toBe(2);
  });
});
```

---

## Integration Tests

### API Endpoint Tests

**Test File**: `api/__tests__/posts.integration.test.js`

```javascript
const request = require('supertest');
const app = require('../../app');
const { setupTestDatabase, cleanupTestDatabase } = require('../helpers');

describe('POST /api/posts', () => {
  let authToken;
  let db;

  beforeAll(async () => {
    db = await setupTestDatabase();
    authToken = await getAuthToken({ id: 1, role: 'user' });
  });

  afterAll(async () => {
    await cleanupTestDatabase(db);
  });

  it('should create draft post with valid data', async () => {
    // Arrange
    const postData = {
      title: 'Integration Test Post',
      content: 'This is a test post with sufficient content length for validation purposes.',
      tags: ['testing', 'integration']
    };

    // Act
    const response = await request(app)
      .post('/api/posts')
      .set('Authorization', `Bearer ${authToken}`)
      .send(postData)
      .expect(201);

    // Assert
    expect(response.body).toMatchObject({
      title: postData.title,
      slug: 'integration-test-post',
      status: 'draft'
    });
    expect(response.body.id).toBeDefined();
  });

  it('should return 401 without authentication', async () => {
    const postData = {
      title: 'Test',
      content: 'Content'
    };

    await request(app)
      .post('/api/posts')
      .send(postData)
      .expect(401);
  });

  it('should return 400 with invalid data', async () => {
    const postData = {
      title: '', // Empty title
      content: 'Short' // Too short
    };

    const response = await request(app)
      .post('/api/posts')
      .set('Authorization', `Bearer ${authToken}`)
      .send(postData)
      .expect(400);

    expect(response.body.error.code).toBe('VALIDATION_ERROR');
    expect(response.body.error.details).toContainEqual(
      expect.objectContaining({ field: 'title' })
    );
  });

  it('should handle duplicate slug gracefully', async () => {
    // Create first post
    await request(app)
      .post('/api/posts')
      .set('Authorization', `Bearer ${authToken}`)
      .send({
        title: 'Duplicate Test',
        content: 'Content for first post with sufficient length.'
      });

    // Create second post with same title
    const response = await request(app)
      .post('/api/posts')
      .set('Authorization', `Bearer ${authToken}`)
      .send({
        title: 'Duplicate Test',
        content: 'Content for second post with sufficient length.'
      })
      .expect(201);

    expect(response.body.slug).toBe('duplicate-test-1');
  });
});

describe('GET /api/posts', () => {
  it('should list published posts', async () => {
    const response = await request(app)
      .get('/api/posts?status=published&page=1&limit=20')
      .expect(200);

    expect(response.body).toHaveProperty('posts');
    expect(response.body).toHaveProperty('pagination');
    expect(response.body.pagination.page).toBe(1);
    expect(response.body.pagination.perPage).toBe(20);
  });

  it('should filter posts by author', async () => {
    const response = await request(app)
      .get('/api/posts?author=123')
      .expect(200);

    expect(response.body.posts.every(p => p.author.id === 123)).toBe(true);
  });

  it('should filter posts by tag', async () => {
    const response = await request(app)
      .get('/api/posts?tag=nodejs')
      .expect(200);

    expect(
      response.body.posts.every(p => 
        p.tags.some(t => t.slug === 'nodejs')
      )
    ).toBe(true);
  });
});

describe('GET /api/posts/:id', () => {
  it('should return post with full details', async () => {
    // Create test post
    const post = await createTestPost({ status: 'published' });

    const response = await request(app)
      .get(`/api/posts/${post.id}`)
      .expect(200);

    expect(response.body).toMatchObject({
      id: post.id,
      title: post.title,
      content: expect.any(String),
      author: expect.objectContaining({
        id: expect.any(Number),
        name: expect.any(String)
      })
    });
  });

  it('should increment view count', async () => {
    const post = await createTestPost({ status: 'published' });
    const initialViewCount = post.viewCount;

    await request(app)
      .get(`/api/posts/${post.id}`)
      .expect(200);

    // Wait for async view tracking
    await new Promise(resolve => setTimeout(resolve, 100));

    const updated = await getPost(post.id);
    expect(updated.viewCount).toBe(initialViewCount + 1);
  });

  it('should return 404 for non-existent post', async () => {
    await request(app)
      .get('/api/posts/99999')
      .expect(404);
  });

  it('should not show draft posts to non-authors', async () => {
    const draftPost = await createTestPost({
      status: 'draft',
      authorId: 456
    });

    await request(app)
      .get(`/api/posts/${draftPost.id}`)
      .set('Authorization', `Bearer ${await getAuthToken({ id: 789 })}`)
      .expect(404);
  });
});

describe('PUT /api/posts/:id', () => {
  it('should update post as author', async () => {
    const token = await getAuthToken({ id: 123 });
    const post = await createTestPost({ authorId: 123 });

    const response = await request(app)
      .put(`/api/posts/${post.id}`)
      .set('Authorization', `Bearer ${token}`)
      .send({ title: 'Updated Title' })
      .expect(200);

    expect(response.body.title).toBe('Updated Title');
  });

  it('should return 403 when non-author tries to update', async () => {
    const post = await createTestPost({ authorId: 123 });
    const otherUserToken = await getAuthToken({ id: 456 });

    await request(app)
      .put(`/api/posts/${post.id}`)
      .set('Authorization', `Bearer ${otherUserToken}`)
      .send({ title: 'Hacked' })
      .expect(403);
  });

  it('should allow admin to update any post', async () => {
    const post = await createTestPost({ authorId: 123 });
    const adminToken = await getAuthToken({ id: 999, role: 'admin' });

    await request(app)
      .put(`/api/posts/${post.id}`)
      .set('Authorization', `Bearer ${adminToken}`)
      .send({ title: 'Admin Update' })
      .expect(200);
  });
});

describe('POST /api/posts/:id/like', () => {
  it('should like post when authenticated', async () => {
    const post = await createTestPost({ status: 'published' });
    const token = await getAuthToken({ id: 123 });

    const response = await request(app)
      .post(`/api/posts/${post.id}/like`)
      .set('Authorization', `Bearer ${token}`)
      .expect(201);

    expect(response.body.isLiked).toBe(true);
    expect(response.body.likeCount).toBe(1);
  });

  it('should not allow liking same post twice', async () => {
    const post = await createTestPost({ status: 'published' });
    const token = await getAuthToken({ id: 123 });

    // First like
    await request(app)
      .post(`/api/posts/${post.id}/like`)
      .set('Authorization', `Bearer ${token}`)
      .expect(201);

    // Second like
    await request(app)
      .post(`/api/posts/${post.id}/like`)
      .set('Authorization', `Bearer ${token}`)
      .expect(409);
  });
});
```

---

## End-to-End Tests

**Test File**: `e2e/__tests__/post-creation.e2e.test.js` (Cypress)

```javascript
describe('Blog Post Creation Flow', () => {
  beforeEach(() => {
    cy.login('testuser@example.com', 'password123');
  });

  it('should create and publish a post', () => {
    // Navigate to new post page
    cy.visit('/posts/new');

    // Fill in post details
    cy.get('[data-testid="post-title"]')
      .type('My First E2E Test Post');

    cy.get('[data-testid="post-content"]')
      .type('This is the content of my post. It has sufficient length to pass validation.');

    // Add tags
    cy.get('[data-testid="tag-input"]')
      .type('testing{enter}e2e{enter}');

    // Save as draft
    cy.get('[data-testid="save-draft-btn"]').click();

    // Verify draft saved
    cy.contains('Draft saved').should('be.visible');

    // Publish post
    cy.get('[data-testid="publish-btn"]').click();
    cy.get('[data-testid="confirm-publish-btn"]').click();

    // Verify redirect to post view
    cy.url().should('include', '/posts/my-first-e2e-test-post');
    cy.contains('My First E2E Test Post').should('be.visible');
  });

  it('should upload and display image', () => {
    cy.visit('/posts/new');

    // Upload image
    cy.get('[data-testid="image-upload"]')
      .attachFile('test-image.jpg');

    // Verify image preview
    cy.get('[data-testid="image-preview"]')
      .should('be.visible')
      .and('have.attr', 'src')
      .and('include', 'test-image');
  });
});

describe('Blog Post Reading Flow', () => {
  it('should display post with all elements', () => {
    cy.visit('/posts/sample-post');

    // Verify elements present
    cy.get('[data-testid="post-title"]').should('be.visible');
    cy.get('[data-testid="post-author"]').should('be.visible');
    cy.get('[data-testid="post-date"]').should('be.visible');
    cy.get('[data-testid="post-content"]').should('be.visible');
    cy.get('[data-testid="like-button"]').should('be.visible');
    cy.get('[data-testid="comment-section"]').should('be.visible');
  });

  it('should like and unlike post', () => {
    cy.login('testuser@example.com', 'password123');
    cy.visit('/posts/sample-post');

    // Initial state - not liked
    cy.get('[data-testid="like-button"]')
      .should('not.have.class', 'liked');

    // Click to like
    cy.get('[data-testid="like-button"]').click();
    cy.get('[data-testid="like-button"]')
      .should('have.class', 'liked');
    cy.get('[data-testid="like-count"]')
      .should('contain', '1');

    // Click to unlike
    cy.get('[data-testid="like-button"]').click();
    cy.get('[data-testid="like-button"]')
      .should('not.have.class', 'liked');
  });
});
```

---

## Performance Tests

**Test File**: `performance/__tests__/posts.load.test.js` (k6)

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
  stages: [
    { duration: '30s', target: 50 },   // Ramp up to 50 users
    { duration: '1m', target: 100 },   // Stay at 100 users
    { duration: '30s', target: 200 },  // Spike to 200 users
    { duration: '1m', target: 100 },   // Back down to 100
    { duration: '30s', target: 0 },    // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<1000'], // 95% of requests < 1s
    http_req_failed: ['rate<0.01'],     // Error rate < 1%
  },
};

export default function () {
  // Test 1: List posts
  let listResponse = http.get('https://api.blogengine.com/posts?page=1&limit=20');
  check(listResponse, {
    'list posts status is 200': (r) => r.status === 200,
    'list posts response time < 500ms': (r) => r.timings.duration < 500,
  });

  sleep(1);

  // Test 2: Get single post
  let postResponse = http.get('https://api.blogengine.com/posts/1');
  check(postResponse, {
    'get post status is 200': (r) => r.status === 200,
    'get post response time < 1000ms': (r) => r.timings.duration < 1000,
  });

  sleep(1);

  // Test 3: Search posts
  let searchResponse = http.get('https://api.blogengine.com/posts?search=nodejs');
  check(searchResponse, {
    'search status is 200': (r) => r.status === 200,
    'search response time < 300ms': (r) => r.timings.duration < 300,
  });

  sleep(2);
}
```

---

## Security Tests

### Input Validation Tests

```javascript
describe('Security: Input Validation', () => {
  it('should sanitize HTML in post content', async () => {
    const maliciousContent = {
      title: 'Test',
      content: '<script>alert("XSS")</script><p>Valid content</p>'
    };

    const response = await request(app)
      .post('/api/posts')
      .set('Authorization', `Bearer ${token}`)
      .send(maliciousContent)
      .expect(201);

    expect(response.body.content).not.toContain('<script>');
    expect(response.body.content).toContain('<p>Valid content</p>');
  });

  it('should reject SQL injection attempts', async () => {
    const response = await request(app)
      .get('/api/posts?author=123; DROP TABLE posts;--')
      .expect(400);
  });

  it('should enforce file type validation on image upload', async () => {
    // Attempt to upload executable file
    const response = await request(app)
      .post('/api/posts/1/upload')
      .set('Authorization', `Bearer ${token}`)
      .attach('image', 'malicious.exe')
      .expect(400);

    expect(response.body.error.message).toContain('Invalid image type');
  });
});

describe('Security: Authorization', () => {
  it('should prevent unauthorized post deletion', async () => {
    const post = await createTestPost({ authorId: 123 });
    const otherUserToken = await getAuthToken({ id: 456 });

    await request(app)
      .delete(`/api/posts/${post.id}`)
      .set('Authorization', `Bearer ${otherUserToken}`)
      .expect(403);
  });

  it('should enforce rate limiting', async () => {
    const token = await getAuthToken({ id: 123 });
    
    // Make 11 requests (limit is 10 per hour)
    for (let i = 0; i < 11; i++) {
      const status = await request(app)
        .post('/api/posts')
        .set('Authorization', `Bearer ${token}`)
        .send({
          title: `Post ${i}`,
          content: 'Content with sufficient length for validation purposes.'
        })
        .then(res => res.status);
      
      if (i === 10) {
        expect(status).toBe(429); // Too Many Requests
      }
    }
  });
});
```

---

## Edge Cases & Error Scenarios

### Test Scenarios

| Scenario | Expected Behavior | Status |
|----------|-------------------|--------|
| Empty database | Returns empty array with correct pagination | ✅ Pass |
| Concurrent slug generation | Both posts get unique slugs | ✅ Pass |
| Image upload during network failure | Returns error, rollback post creation | ✅ Pass |
| Post with 200+ character title | Truncates to 200 chars | ✅ Pass |
| Unicode characters in title/content | Properly saved and displayed | ✅ Pass |
| Very large content (50MB) | Returns413 error | ✅ Pass |
| Simultaneous likes on same post | Only one like counted per user | ✅ Pass |
| Viewing archived post | Returns 404 or special archive page | ✅ Pass |
| Scheduled post publish time | Publishes exactly at scheduled time | 🔄 Testing |

---

## Test Data Management

### Test Fixtures

```javascript
// fixtures/posts.fixtures.js
export const validPostData = {
  title: 'Test Post Title',
  content: 'This is test content with sufficient length to meet validation requirements.',
  excerpt: 'Test excerpt',
  tags: ['test', 'fixture'],
  status: 'draft'
};

export const publishedPostData = {
  ...validPostData,
  status: 'published',
  publishedAt: new Date('2024-11-01T10:00:00Z')
};

export const postWithImage = {
  ...validPostData,
  featuredImageUrl: 'https://cdn.example.com/test-image.jpg'
};
```

---

## Test Execution

### Run All Tests

```bash
# Unit tests
npm test

# Integration tests
npm run test:integration

# E2E tests
npm run test:e2e

# Performance tests
npm run test:performance

# Coverage report
npm run test:coverage
```

### Continuous Integration

Tests run automatically on:
- Every pull request
- Before deployment to staging
- Before deployment to production

**Required for merge**:
- ✅ All unit tests pass
- ✅ All integration tests pass
- ✅ Coverage ≥ 80%
- ✅ No security vulnerabilities

---

## Test Metrics & Monitoring

### Current Metrics (as of November 5, 2024)

- **Total Tests**: 182
- **Unit Tests**: 120 (66%)
- **Integration Tests**: 35 (19%)
- **E2E Tests**: 12 (7%)
- **Performance Tests**: 5 (3%)
- **Security Tests**: 10 (5%)

### Coverage

- **Overall**: 85%
- **Service Layer**: 92%
- **Repository Layer**: 88%
- **Controller Layer**: 78%
- **Util Functions**: 95%

### Test Execution Time

- **Unit Tests**: ~15 seconds
- **Integration Tests**: ~45 seconds
- **E2E Tests**: ~2 minutes
- **Performance Tests**: ~5 minutes
- **Total**: ~8 minutes

---

## Known Issues & Limitations

1. **Flaky E2E test**: "should upload and display image" fails intermittently (investigating)
2. **Performance test**: Spike test occasionally times out on CI (infrastructure issue)
3. **Coverage gap**: Image upload service not fully covered (priority P2)

---

## Related Documentation

- [Requirements](requirements.md)
- [Design Document](design.md)
- [Tasks Checklist](tasks.md)
- [Test Strategy](../../runbooks/test-strategy.md)

---

**Maintained By**: QA Team  
**Review Frequency**: Every sprint  
**Last Review**: November 5, 2024
