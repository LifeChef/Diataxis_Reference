# Database Schema Reference

> **Document Type**: Reference (Information-Oriented)  
> **Category**: Database  
> **Version**: 1.0  
> **Last Updated**: November 5, 2024

## Overview

Complete database schema for Blog Engine PostgreSQL database.

**Database**: PostgreSQL 14+  
**Schema Version**: 1.0  
**Total Tables**: 8

---

## Entity Relationship Diagram

```
┌─────────────┐     ┌─────────────┐     ┌──────────────┐
│    users    │────<│    posts    │>────│   comments   │
└─────────────┘     └─────────────┘     └──────────────┘
       │                   │                     │
       │                   │                     │
       │            ┌──────┴──────┐             │
       │            │              │             │
       ▼            ▼              ▼             ▼
┌─────────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│notifications│  │post_tags │  │post_likes│  │  comment │
└─────────────┘  └──────────┘  └──────────┘  │  _likes  │
                                               └──────────┘
```

---

## Tables

### users

Stores user account information.

**Columns**:
| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| username | VARCHAR(20) | No | - | Unique username |
| email | VARCHAR(255) | No | - | Unique email |
| password_hash | VARCHAR(255) | No | - | Bcrypt hashed password |
| display_name | VARCHAR(100) | Yes | - | Display name |
| bio | TEXT | Yes | - | User biography |
| avatar_url | VARCHAR(500) | Yes | - | Profile picture URL |
| role | VARCHAR(20) | No | 'user' | user, moderator, admin |
| email_verified | BOOLEAN | No | false | Email verification status |
| is_active | BOOLEAN | No | true | Account active status |
| created_at | TIMESTAMP | No | NOW() | Account creation time |
| updated_at | TIMESTAMP | No | NOW() | Last update time |

**Indexes**:
- PRIMARY KEY: `id`
- UNIQUE: `username`, `email`
- INDEX: `created_at`, `role`

**Constraints**:
- `CHECK (char_length(username) >= 3)`
- `CHECK (char_length(password_hash) > 0)`
- `CHECK (role IN ('user', 'moderator', 'admin'))`

---

### posts

Stores blog posts.

**Columns**:
| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| author_id | UUID | No | - | Foreign key to users.id |
| title | VARCHAR(200) | No | - | Post title |
| slug | VARCHAR(250) | No | - | URL-friendly slug |
| content | TEXT | No | - | Markdown content |
| excerpt | VARCHAR(500) | Yes | - | Short excerpt |
| cover_image_url | VARCHAR(500) | Yes | - | Cover image URL |
| category | VARCHAR(50) | No | - | Post category |
| status | VARCHAR(20) | No | 'draft' | draft, published, archived |
| views | INTEGER | No | 0 | View count |
| published_at | TIMESTAMP | Yes | - | Publication timestamp |
| created_at | TIMESTAMP | No | NOW() | Creation timestamp |
| updated_at | TIMESTAMP | No | NOW() | Last update timestamp |

**Indexes**:
- PRIMARY KEY: `id`
- UNIQUE: `slug`
- FOREIGN KEY: `author_id` REFERENCES `users(id)` ON DELETE CASCADE
- INDEX: `author_id`, `status`, `published_at`, `category`, `created_at`

**Constraints**:
- `CHECK (char_length(title) >= 3)`
- `CHECK (status IN ('draft', 'published', 'archived'))`
- `CHECK (views >= 0)`

---

### comments

Stores post comments (supports threaded replies).

**Columns**:
| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| post_id | UUID | No | - | Foreign key to posts.id |
| author_id | UUID | No | - | Foreign key to users.id |
| parent_id | UUID | Yes | - | Parent comment (for replies) |
| content | TEXT | No | - | Comment text |
| is_moderated | BOOLEAN | No | false | Moderation flag |
| is_flagged | BOOLEAN | No | false | User flagged |
| created_at | TIMESTAMP | No | NOW() | Creation timestamp |
| updated_at | TIMESTAMP | No | NOW() | Last update timestamp |

**Indexes**:
- PRIMARY KEY: `id`
- FOREIGN KEY: `post_id` REFERENCES `posts(id)` ON DELETE CASCADE
- FOREIGN KEY: `author_id` REFERENCES `users(id)` ON DELETE CASCADE
- FOREIGN KEY: `parent_id` REFERENCES `comments(id)` ON DELETE CASCADE
- INDEX: `post_id`, `author_id`, `parent_id`, `created_at`

**Constraints**:
- `CHECK (char_length(content) >= 1 AND char_length(content) <= 2000)`

---

### post_tags

Many-to-many relationship between posts and tags.

**Columns**:
| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| post_id | UUID | No | - | Foreign key to posts.id |
| tag | VARCHAR(50) | No | - | Tag name |
| created_at | TIMESTAMP | No | NOW() | Creation timestamp |

**Indexes**:
- PRIMARY KEY: `(post_id, tag)`
- FOREIGN KEY: `post_id` REFERENCES `posts(id)` ON DELETE CASCADE
- INDEX: `tag`, `created_at`

**Constraints**:
- `CHECK (char_length(tag) >= 2)`

---

### post_likes

Tracks user likes on posts.

**Columns**:
| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| post_id | UUID | No | - | Foreign key to posts.id |
| user_id | UUID | No | - | Foreign key to users.id |
| created_at | TIMESTAMP | No | NOW() | Like timestamp |

**Indexes**:
- PRIMARY KEY: `(post_id, user_id)`
- FOREIGN KEY: `post_id` REFERENCES `posts(id)` ON DELETE CASCADE
- FOREIGN KEY: `user_id` REFERENCES `users(id)` ON DELETE CASCADE
- INDEX: `user_id`, `created_at`

---

### comment_likes

Tracks user likes on comments.

**Columns**:
| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| comment_id | UUID | No | - | Foreign key to comments.id |
| user_id | UUID | No | - | Foreign key to users.id |
| created_at | TIMESTAMP | No | NOW() | Like timestamp |

**Indexes**:
- PRIMARY KEY: `(comment_id, user_id)`
- FOREIGN KEY: `comment_id` REFERENCES `comments(id)` ON DELETE CASCADE
- FOREIGN KEY: `user_id` REFERENCES `users(id)` ON DELETE CASCADE

---

### notifications

Stores user notifications.

**Columns**:
| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| user_id | UUID | No | - | Recipient user ID |
| type | VARCHAR(50) | No | - | Notification type |
| title | VARCHAR(200) | No | - | Notification title |
| message | TEXT | No | - | Notification message |
| link | VARCHAR(500) | Yes | - | Related link |
| is_read | BOOLEAN | No | false | Read status |
| created_at | TIMESTAMP | No | NOW() | Creation timestamp |

**Indexes**:
- PRIMARY KEY: `id`
- FOREIGN KEY: `user_id` REFERENCES `users(id)` ON DELETE CASCADE
- INDEX: `user_id`, `is_read`, `created_at`

**Notification Types**:
- `new_comment`: New comment on user's post
- `comment_reply`: Reply to user's comment
- `post_like`: Someone liked user's post
- `new_follower`: New follower
- `mention`: User mentioned in post/comment

---

### sessions

Stores active user sessions (for JWT token management).

**Columns**:
| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| user_id | UUID | No | - | Foreign key to users.id |
| refresh_token | VARCHAR(500) | No | - | Refresh token |
| expires_at | TIMESTAMP | No | - | Expiration timestamp |
| created_at | TIMESTAMP | No | NOW() | Creation timestamp |

**Indexes**:
- PRIMARY KEY: `id`
- FOREIGN KEY: `user_id` REFERENCES `users(id)` ON DELETE CASCADE
- INDEX: `user_id`, `refresh_token`, `expires_at`

---

## Migrations

Migrations are located in `/migrations` directory.

**Running Migrations**:
```bash
# Run all pending migrations
npm run migrate:latest

# Rollback last migration
npm run migrate:rollback

# Create new migration
npm run migrate:create -- migration_name
```

---

## Data Retention

**Policies**:
- Deleted user accounts: 30-day grace period, then permanent deletion
- Archived posts: Retained indefinitely
- Audit logs: 1 year retention
- Sessions: Auto-deleted on expiry

---

## Performance Considerations

**Connection Pooling**:
- Min connections: 10
- Max connections: 100
- Idle timeout: 30 seconds

**Query Optimization**:
- All foreign keys indexed
- Compound indexes on frequently queried columns
- Partial indexes on status fields

**Caching Strategy**:
- Hot data (popular posts, user profiles) cached in Redis
- Cache TTL: 5 minutes
- Cache invalidation on updates

---

**See Also**:
- [System Architecture](../architecture/system-architecture.md)
- [Configuration Parameters](../configuration/parameters.md)

---

**Maintained By**: Database Team  
**Last Reviewed**: November 5, 2024
