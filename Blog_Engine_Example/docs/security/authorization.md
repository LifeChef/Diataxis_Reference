# Authorization

> **Document Type**: Security Control  
> **SOC2 Controls**: CC6.1, CC6.2, CC6.3  
> **Last Updated**: November 5, 2024  
> **Status**: ✅ Implemented

---

## Overview

This document defines the authorization model for the Blog Engine system using Role-Based Access Control (RBAC) with resource-level permissions.

---

## Authorization Model

### RBAC (Role-Based Access Control)

Users are assigned roles, and roles have permissions.

```
User → Role → Permissions → Resources
```

---

## User Roles

| Role | Description | Typical Users |
|------|-------------|---------------|
| **viewer** | Read-only access | Anonymous visitors |
| **user** | Create and manage own content | Registered users |
| **moderator** | Moderate content, manage users | Community managers |
| **admin** | Full system access | System administrators |
| **super_admin** | All permissions + system config | CTO, Lead Engineer |

---

## Permissions Matrix

### Post Permissions

| Action | Viewer | User | Moderator | Admin |
|--------|--------|------|-----------|-------|
| Read published posts | ✅ | ✅ | ✅ | ✅ |
| Read own drafts | ❌ | ✅ | ✅ | ✅ |
| Read all drafts | ❌ | ❌ | ✅ | ✅ |
| Create post | ❌ | ✅ | ✅ | ✅ |
| Edit own post | ❌ | ✅ | ✅ | ✅ |
| Edit any post | ❌ | ❌ | ✅ | ✅ |
| Delete own post | ❌ | ✅ | ✅ | ✅ |
| Delete any post | ❌ | ❌ | ✅ | ✅ |
| Publish post | ❌ | ✅ | ✅ | ✅ |

### User Management Permissions

| Action | Viewer | User | Moderator | Admin |
|--------|--------|------|-----------|-------|
| View own profile | ❌ | ✅ | ✅ | ✅ |
| Edit own profile | ❌ | ✅ | ✅ | ✅ |
| View user list | ❌ | ❌ | ✅ | ✅ |
| Suspend users | ❌ | ❌ | ✅ | ✅ |
| Delete users | ❌ | ❌ | ❌ | ✅ |
| Change user roles | ❌ | ❌ | ❌ | ✅ |

### Comment Permissions

| Action | Viewer | User | Moderator | Admin |
|--------|--------|------|-----------|-------|
| Read comments | ✅ | ✅ | ✅ | ✅ |
| Create comment | ❌ | ✅ | ✅ | ✅ |
| Edit own comment | ❌ | ✅ | ✅ | ✅ |
| Delete own comment | ❌ | ✅ | ✅ | ✅ |
| Delete any comment | ❌ | ❌ | ✅ | ✅ |
| Flag comment | ❌ | ✅ | ✅ | ✅ |

---

## Implementation

### Permission Definitions

```javascript
// permissions.js
const PERMISSIONS = {
  // Posts
  'post:read': ['viewer', 'user', 'moderator', 'admin'],
  'post:create': ['user', 'moderator', 'admin'],
  'post:update:own': ['user', 'moderator', 'admin'],
  'post:update:any': ['moderator', 'admin'],
  'post:delete:own': ['user', 'moderator', 'admin'],
  'post:delete:any': ['moderator', 'admin'],
  'post:publish': ['user', 'moderator', 'admin'],
  
  // Users
  'user:read:own': ['user', 'moderator', 'admin'],
  'user:read:any': ['moderator', 'admin'],
  'user:update:own': ['user', 'moderator', 'admin'],
  'user:delete:any': ['admin'],
  'user:role:change': ['admin'],
  
  // Comments
  'comment:create': ['user', 'moderator', 'admin'],
  'comment:update:own': ['user', 'moderator', 'admin'],
  'comment:delete:own': ['user', 'moderator', 'admin'],
  'comment:delete:any': ['moderator', 'admin'],
  
  // Admin
  'admin:analytics': ['admin'],
  'admin:settings': ['admin'],
  'admin:logs': ['admin']
};

module.exports = PERMISSIONS;
```

### Authorization Middleware

```javascript
// middleware/authorize.js
const PERMISSIONS = require('../permissions');

/**
 * Check if user has permission
 */
function authorize(permission) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({
        error: {
          code: 'UNAUTHORIZED',
          message: 'Authentication required'
        }
      });
    }

    const allowedRoles = PERMISSIONS[permission];
    
    if (!allowedRoles || !allowedRoles.includes(req.user.role)) {
      return res.status(403).json({
        error: {
          code: 'FORBIDDEN',
          message: 'Insufficient permissions',
          details: [{
            required: permission,
            userRole: req.user.role
          }]
        }
      });
    }

    next();
  };
}

module.exports = authorize;
```

### Usage in Routes

```javascript
const authorize = require('./middleware/authorize');

// Public endpoint - no auth required
app.get('/api/posts', async (req, res) => {
  const posts = await postService.getPublishedPosts();
  res.json(posts);
});

// Requires authentication and permission
app.post('/api/posts',
  authenticate,
  authorize('post:create'),
  async (req, res) => {
    const post = await postService.createPost(req.user.id, req.body);
    res.status(201).json(post);
  }
);

// Resource-level authorization
app.put('/api/posts/:id',
  authenticate,
  async (req, res) => {
    const post = await postService.getById(req.params.id);
    
    if (!post) {
      return res.status(404).json({ error: 'Post not found' });
    }
    
    // Check ownership or moderator role
    const canUpdate = 
      post.authorId === req.user.id || // Own post
      ['moderator', 'admin'].includes(req.user.role); // Or has role
    
    if (!canUpdate) {
      return res.status(403).json({
        error: {
          code: 'FORBIDDEN',
          message: 'Cannot update this post'
        }
      });
    }
    
    const updated = await postService.update(req.params.id, req.body);
    res.json(updated);
  }
);
```

---

## Resource Ownership

### Ownership Rules

Users can **always** manage their own resources:
- **Posts**: Author can edit/delete own posts
- **Comments**: User can edit/delete own comments
- **Profile**: User can edit own profile

**Exception**: Moderators and admins can manage any resource

### Ownership Check Helper

```javascript
function canAccessResource(user, resource, action) {
  // Super admin can do anything
  if (user.role === 'super_admin') {
    return true;
  }
  
  // Admin can do most things
  if (user.role === 'admin' && action !== 'delete:system') {
    return true;
  }
  
  // Moderator can manage content
  if (user.role === 'moderator' && 
      ['post', 'comment'].includes(resource.type)) {
    return true;
  }
  
  // Owner can manage own resource
  if (resource.ownerId === user.id) {
    return true;
  }
  
  return false;
}
```

---

## Special Cases

### Soft Delete vs Hard Delete

```javascript
// Soft delete - users can delete own posts
app.delete('/api/posts/:id',
  authenticate,
  async (req, res) => {
    const post = await postService.getById(req.params.id);
    
    if (post.authorId !== req.user.id && req.user.role !== 'admin') {
      return res.status(403).json({ error: 'Forbidden' });
    }
    
    // Soft delete (set deleted_at)
    await postService.softDelete(req.params.id);
    res.status(204).send();
  }
);

// Hard delete - admin only
app.delete('/api/admin/posts/:id/permanent',
  authenticate,
  authorize('admin'),
  async (req, res) => {
    await postService.hardDelete(req.params.id);
    res.status(204).send();
  }
);
```

### Delegation

Users can delegate permissions to others (future feature):

```javascript
// Grant user temporary moderator access to specific post
await db('post_collaborators').insert({
  post_id: postId,
  user_id: collaboratorId,
  role: 'editor',
  granted_by: req.user.id,
  expires_at: new Date(Date.now() + 7 * 24 * 3600000) // 7 days
});
```

---

## API Key Authorization

For programmatic access:

```javascript
// API key middleware
async function authorizeApiKey(req, res, next) {
  const apiKey = req.headers['x-api-key'];
  
  if (!apiKey) {
    return res.status(401).json({ error: 'API key required' });
  }
  
  const keyData = await db('api_keys')
    .where('key_hash', hashApiKey(apiKey))
    .where('revoked', false)
    .where('expires_at', '>', new Date())
    .first();
  
  if (!keyData) {
    return res.status(401).json({ error: 'Invalid API key' });
  }
  
  // Load user and permissions
  req.user = await db('users').where('id', keyData.user_id).first();
  req.apiKey = keyData;
  
  next();
}

// API-specific permissions
const API_PERMISSIONS = {
  'api_key:read_only': ['post:read', 'user:read:own'],
  'api_key:full_access': ['post:*', 'comment:*']
};
```

---

## Audit Logging

Log all authorization decisions:

```javascript
function logAuthorizationCheck(user, resource, action, granted) {
  logger.info('Authorization check', {
    user: {
      id: user.id,
      role: user.role
    },
    resource: {
      type: resource.type,
      id: resource.id,
      ownerId: resource.ownerId
    },
    action,
    granted,
    timestamp: new Date().toISOString()
  });
}
```

---

## Testing Authorization

```javascript
describe('Post Authorization', () => {
  it('should allow user to edit own post', async () => {
    const user = { id: 123, role: 'user' };
    const post = { id: 1, authorId: 123 };
    
    const canUpdate = canAccessResource(user, { 
      type: 'post', 
      ownerId: post.authorId 
    }, 'update');
    
    expect(canUpdate).toBe(true);
  });
  
  it('should deny user from editing others post', async () => {
    const user = { id: 123, role: 'user' };
    const post = { id: 1, authorId: 456 };
    
    const canUpdate = canAccessResource(user, { 
      type: 'post', 
      ownerId: post.authorId 
    }, 'update');
    
    expect(canUpdate).toBe(false);
  });
  
  it('should allow moderator to edit any post', async () => {
    const user = { id: 123, role: 'moderator' };
    const post = { id: 1, authorId: 456 };
    
    const canUpdate = canAccessResource(user, { 
      type: 'post', 
      ownerId: post.authorId 
    }, 'update');
    
    expect(canUpdate).toBe(true);
  });
});
```

---

## Best Practices

### DO ✅

- Always check authorization after authentication
- Use resource-level permissions for sensitive data
- Log all authorization failures
- Implement least privilege principle
- Validate ownership for user resources
- Use role hierarchy appropriately

### DON'T ❌

- Trust client-side authorization
- Expose unauthorized data in API responses
- Hard-code permissions in multiple places
- Allow privilege escalation
- Skip authorization checks for "internal" endpoints

---

## Related Documentation

- [Authentication Security](authentication.md)
- [API Security](api-security.md)
- [Data Protection](data-protection.md)

---

**Security Contact**: security@blogengine.com  
**Last Review**: November 5, 2024
