# Implement a New API Endpoint

> **Document Type**: How-To Guide (SAMPLE)  
> **Audience**: Backend Developers  
> **Difficulty**: Beginner  
> **Last Updated**: November 5, 2024

---

## Problem

You need to create a new REST API endpoint following the Blog Engine conventions and best practices.

---

## Prerequisites

- Local development environment set up
- Understanding of Express.js
- Familiarity with the service architecture

---

## Solution

### Step 1: Define the Endpoint Specification

Document in `/docs/api/rest-api.md`:

```markdown
## GET /api/posts/:id/likes

Get all likes for a specific post.

**Authentication**: Required

**Parameters**:
- `id` (path): Post ID

**Response** (200 OK):
```json
{
  "likes": [
    { "userId": 123, "createdAt": "November 5, 2024T10:30:00Z" }
  ],
  "total": 42
}
```

### Step 2: Create the Controller

Create `src/controllers/post-likes.controller.js`:

```javascript
const PostLikesService = require('../services/post-likes.service');

class PostLikesController {
  async getLikes(req, res, next) {
    try {
      const { id } = req.params;
      const likes = await PostLikesService.getLikesByPostId(id);
      
      res.json({
        likes: likes,
        total: likes.length
      });
    } catch (error) {
      next(error);
    }
  }
  
  async addLike(req, res, next) {
    try {
      const { id } = req.params;
      const userId = req.user.id;
      
      const like = await PostLikesService.addLike(id, userId);
      
      res.status(201).json(like);
    } catch (error) {
      next(error);
    }
  }
}

module.exports = new PostLikesController();
```

### Step 3: Create the Service

Create `src/services/post-likes.service.js`:

```javascript
const db = require('../db');

class PostLikesService {
  async getLikesByPostId(postId) {
    const likes = await db('post_likes')
      .where({ post_id: postId })
      .select('user_id as userId', 'created_at as createdAt');
    
    return likes;
  }
  
  async addLike(postId, userId) {
    // Check if already liked
    const existing = await db('post_likes')
      .where({ post_id: postId, user_id: userId })
      .first();
    
    if (existing) {
      throw new Error('Post already liked by user');
    }
    
    // Add like
    const [like] = await db('post_likes')
      .insert({ post_id: postId, user_id: userId })
      .returning('*');
    
    return like;
  }
}

module.exports = new PostLikesService();
```

### Step 4: Add Routes

Update `src/routes/posts.routes.js`:

```javascript
const express = require('express');
const router = express.Router();
const PostLikesController = require('../controllers/post-likes.controller');
const authMiddleware = require('../middleware/auth.middleware');

// Get likes for a post
router.get('/:id/likes', PostLikesController.getLikes);

// Add like to a post
router.post('/:id/likes', authMiddleware, PostLikesController.addLike);

module.exports = router;
```

### Step 5: Add Validation

Create `src/validators/post-likes.validator.js`:

```javascript
const { param } = require('express-validator');

const validatePostId = [
  param('id').isInt().withMessage('Post ID must be an integer')
];

module.exports = { validatePostId };
```

### Step 6: Write Tests

Create `src/__tests__/post-likes.test.js`:

```javascript
const request = require('supertest');
const app = require('../app');

describe('POST /api/posts/:id/likes', () => {
  it('should add like when authenticated', async () => {
    const token = await getAuthToken();
    
    const response = await request(app)
      .post('/api/posts/1/likes')
      .set('Authorization', `Bearer ${token}`)
      .expect(201);
    
    expect(response.body).toHaveProperty('user_id');
  });
  
  it('should return 401 when not authenticated', async () => {
    await request(app)
      .post('/api/posts/1/likes')
      .expect(401);
  });
});
```

### Step 7: Update API Documentation

Update OpenAPI/Swagger spec if used.

---

## Verification

1. Run tests: `npm test`
2. Start server: `npm run dev`
3. Test endpoint manually:

```bash
# Get likes
curl http://localhost:3000/api/posts/1/likes

# Add like (with auth)
curl -X POST http://localhost:3000/api/posts/1/likes \
  -H "Authorization: Bearer YOUR_TOKEN"
```

---

## Best Practices Checklist

- [ ] Follow RESTful conventions
- [ ] Add authentication/authorization
- [ ] Input validation implemented
- [ ] Error handling added
- [ ] Unit tests written
- [ ] Integration tests written
- [ ] API documentation updated
- [ ] Logging added for important actions

---

## Related

- [API Conventions](../../docs/protocols/api-conventions.md)
- [Error Handling Standards](../../docs/protocols/error-handling.md)
- [API Reference](../../reference/api/endpoints.md)
