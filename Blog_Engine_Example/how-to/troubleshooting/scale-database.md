# Scale Database Performance

> **Document Type**: How-To Guide (SAMPLE)  
> **Audience**: DevOps Engineers, DBAs, Backend Developers  
> **Difficulty**: Advanced  
> **Last Updated**: November 5, 2024

---

## Problem

Your PostgreSQL database is experiencing performance issues or high load, and you need to scale it to handle more traffic.

---

## Prerequisites

- Database access with admin privileges
- Understanding of SQL and database concepts
- Monitoring tools configured

---

## Solution

### Step 1: Identify the Bottleneck

**Check current metrics:**

```sql
-- Connection count
SELECT count(*) FROM pg_stat_activity;

-- Slow queries
SELECT query, mean_exec_time, calls
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

-- Table sizes
SELECT schemaname, tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Cache hit ratio (should be > 99%)
SELECT 
  sum(heap_blks_read) as heap_read,
  sum(heap_blks_hit)  as heap_hit,
  sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) as ratio
FROM pg_statio_user_tables;
```

---

### Step 2: Optimize Queries (First Priority)

**Add missing indexes:**

```sql
-- Identify missing indexes
SELECT schemaname, tablename, attname
FROM pg_stats
WHERE schemaname = 'public'
  AND n_distinct > 100
  AND correlation < 0.1
  AND attname NOT IN (
    SELECT a.attname
    FROM pg_index i
    JOIN pg_attribute a ON a.attrelid = i.indrelid
      AND a.attnum = ANY(i.indkey)
  );

-- Create indexes for common queries
CREATE INDEX CONCURRENTLY idx_posts_author_created 
ON posts(author_id, created_at DESC);

CREATE INDEX CONCURRENTLY idx_posts_status_created 
ON posts(status, created_at DESC) 
WHERE status = 'published';

CREATE INDEX CONCURRENTLY idx_comments_post_created 
ON comments(post_id, created_at DESC);
```

**Optimize N+1 queries:**

```javascript
// BEFORE (N+1 query)
const posts = await Post.findAll();
for (const post of posts) {
  post.author = await User.findById(post.authorId); // N queries!
}

// AFTER (Single query with JOIN)
const posts = await Post.findAll({
  include: ['author'] // Single query with JOIN
});
```

---

### Step 3: Increase Connection Pool

**Update application config:**

```javascript
// config/database.js
const pool = new Pool({
  host: process.env.DB_HOST,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  max: 50,              // Increased from 20
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});
```

**Increase PostgreSQL max connections:**

```sql
-- Check current limit
SHOW max_connections; -- Default: 100

-- Increase in postgresql.conf
ALTER SYSTEM SET max_connections = 200;

-- Restart PostgreSQL
-- kubectl rollout restart statefulset/postgres -n database
```

---

### Step 4: Add Read Replicas

**Create read replica (AWS RDS example):**

```bash
aws rds create-db-instance-read-replica \
  --db-instance-identifier blogengine-read-replica-1 \
  --source-db-instance-identifier blogengine-primary \
  --db-instance-class db.t3.medium \
  --availability-zone us-east-1a
```

**Configure application to use read replica:**

```javascript
// config/database.js
const config = {
  write: {
    host: process.env.DB_WRITE_HOST, // Primary
    port: 5432,
    database: 'blogengine'
  },
  read: [
    {
      host: process.env.DB_READ_HOST_1, // Replica 1
      port: 5432,
      database: 'blogengine'
    },
    {
      host: process.env.DB_READ_HOST_2, // Replica 2
      port: 5432,
      database: 'blogengine'
    }
  ],
  pool: { max: 50, min: 5 }
};

// Use in Sequelize
const sequelize = new Sequelize('blogengine', 'user', 'password', {
  dialect: 'postgres',
  replication: config
});

// Read queries automatically go to replicas
const posts = await Post.findAll(); // Uses read replica

// Write queries go to primary
await Post.create({ title: 'New Post' }); // Uses primary
```

---

### Step 5: Implement Caching with Redis

**Install Redis client:**

```bash
npm install redis
```

**Create cache service:**

```javascript
// services/cache.service.js
const redis = require('redis');

class CacheService {
  constructor() {
    this.client = redis.createClient({
      url: process.env.REDIS_URL
    });
    this.client.connect();
  }

  async get(key) {
    const cached = await this.client.get(key);
    return cached ? JSON.parse(cached) : null;
  }

  async set(key, value, ttl = 3600) {
    await this.client.setEx(key, ttl, JSON.stringify(value));
  }

  async del(key) {
    await this.client.del(key);
  }
}

module.exports = new CacheService();
```

**Use caching in controllers:**

```javascript
const cacheService = require('./services/cache.service');

router.get('/api/posts/:id', async (req, res) => {
  const cacheKey = `post:${req.params.id}`;
  
  // Try cache first
  let post = await cacheService.get(cacheKey);
  
  if (!post) {
    // Cache miss - fetch from database
    post = await Post.findById(req.params.id);
    
    // Cache for 1 hour
    await cacheService.set(cacheKey, post, 3600);
  }
  
  res.json(post);
});

// Invalidate cache on update
router.put('/api/posts/:id', async (req, res) => {
  const post = await Post.update(req.params.id, req.body);
  
  // Invalidate cache
  await cacheService.del(`post:${req.params.id}`);
  
  res.json(post);
});
```

---

### Step 6: Implement Database Connection Pooling (PgBouncer)

**Deploy PgBouncer:**

```yaml
# k8s/database/pgbouncer.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pgbouncer
  namespace: database
spec:
  replicas: 2
  selector:
    matchLabels:
      app: pgbouncer
  template:
    metadata:
      labels:
        app: pgbouncer
    spec:
      containers:
      - name: pgbouncer
        image: pgbouncer/pgbouncer:latest
        ports:
        - containerPort: 5432
        env:
        - name: DATABASES_HOST
          value: "postgres.database.svc.cluster.local"
        - name: DATABASES_PORT
          value: "5432"
        - name: DATABASES_USER
          value: "blogengine"
        - name: DATABASES_DBNAME
          value: "blogengine"
        - name: PGBOUNCER_POOL_MODE
          value: "transaction"
        - name: PGBOUNCER_MAX_CLIENT_CONN
          value: "1000"
        - name: PGBOUNCER_DEFAULT_POOL_SIZE
          value: "25"
```

**Update application to use PgBouncer:**

```bash
# Before
DATABASE_URL=postgres://user:pass@postgres:5432/blogengine

# After
DATABASE_URL=postgres://user:pass@pgbouncer:5432/blogengine
```

---

### Step 7: Vertical Scaling (Increase Resources)

**AWS RDS:**

```bash
aws rds modify-db-instance \
  --db-instance-identifier blogengine-primary \
  --db-instance-class db.r5.xlarge \
  --apply-immediately
```

**Self-hosted (Kubernetes):**

```yaml
# Update StatefulSet
spec:
  template:
    spec:
      containers:
      - name: postgres
        resources:
          requests:
            memory: "4Gi"
            cpu: "2000m"
          limits:
            memory: "8Gi"
            cpu: "4000m"
```

---

### Step 8: Database Maintenance

**Regular maintenance tasks:**

```sql
-- Vacuum and analyze (reclaim space, update statistics)
VACUUM ANALYZE posts;
VACUUM ANALYZE comments;

-- Reindex (rebuild indexes for better performance)
REINDEX TABLE posts;
REINDEX TABLE comments;

-- Update table statistics
ANALYZE posts;
ANALYZE comments;
```

**Automate with cron job:**

```yaml
# k8s/database/maintenance-cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: db-maintenance
  namespace: database
spec:
  schedule: "0 2 * * 0"  # Weekly, Sunday 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: maintenance
            image: postgres:14
            command:
            - /bin/sh
            - -c
            - |
              psql $DATABASE_URL <<EOF
                VACUUM ANALYZE;
                REINDEX DATABASE blogengine;
              EOF
          restartPolicy: OnFailure
```

---

## Verification

**Check improvements:**

```sql
-- Query performance
EXPLAIN ANALYZE
SELECT * FROM posts WHERE author_id = 123 ORDER BY created_at DESC LIMIT 10;

-- Cache hit ratio (should improve to > 99%)
SELECT 
  sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) as ratio
FROM pg_statio_user_tables;

-- Connection pool usage
SELECT count(*) as active_connections FROM pg_stat_activity WHERE state = 'active';
```

**Monitor improvements in Grafana dashboards**

---

## Performance Checklist

- [ ] Slow queries identified and optimized
- [ ] Missing indexes added
- [ ] N+1 queries eliminated
- [ ] Connection pool sized appropriately
- [ ] Read replicas configured (if needed)
- [ ] Caching implemented for frequently accessed data
- [ ] PgBouncer deployed (optional)
- [ ] Regular maintenance scheduled
- [ ] Monitoring and alerts configured

---

## Troubleshooting

**High CPU usage:**
- Review slow query log
- Check for missing indexes
- Look for table scans in EXPLAIN plans

**High memory usage:**
- Adjust `shared_buffers` (25% of RAM)
- Reduce `work_mem` if too high
- Check for memory leaks in application

**Connection pool exhausted:**
- Increase pool size in application
- Add PgBouncer
- Check for connection leaks
- Reduce connection timeout

---

## Related

- [Database Schema Reference](../../reference/database/schema.md)
- [Operations Runbook](../../runbooks/operations.md)
- [Monitoring Setup](../deployment/monitoring-setup.md)
- [PostgreSQL Performance Tuning](https://wiki.postgresql.org/wiki/Performance_Optimization)
