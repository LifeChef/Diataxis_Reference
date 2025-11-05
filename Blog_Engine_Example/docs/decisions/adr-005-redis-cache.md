# ADR-005: Redis for Caching

**Status**: ✅ Accepted  
**Date**: October 20, 2024  
**Deciders**: Architecture Team (Sarah Johnson, Mike Chen, David Kim)  
**Last Updated**: November 5, 2024

---

## Context

The Blog Engine application needs a caching layer to:
- Reduce database load from repeated queries
- Improve API response times
- Handle high read traffic (10:1 read/write ratio)
- Support session storage
- Enable rate limiting

### Performance Goals

- **API Response Time**: < 100ms for cached responses (vs 500ms DB queries)
- **Cache Hit Rate**: > 80%
- **Read Reduction**: 70% fewer database reads
- **Throughput**: Support 1,000 requests/second

### Use Cases

1. **Data Caching**: Post details, user profiles, tag lists
2. **Session Storage**: User sessions, JWT tokens
3. **Rate Limiting**: Track API request counts
4. **Temporary Data**: Password reset tokens, email verification codes
5. **Pub/Sub**: Real-time notifications (secondary use)

---

## Decision

We will use **Redis** as our primary caching layer.

### Architecture

```
┌─────────┐           ┌─────────┐           ┌─────────────┐
│  Client │ ──req──> │   API   │ ─check──> │    Redis    │
│         │           │ Service │           │   (Cache)   │
└─────────┘           └────┬────┘           └─────────────┘
                           │                        │
                           │                      miss
                           │                        │
                           │               ┌────────▼────────┐
                           └─────query───> │   PostgreSQL    │
                                           │   (Database)    │
                                           └─────────────────┘
```

### Configuration

- **Provider**: AWS ElastiCache for Redis
- **Version**: Redis 7.0
- **Mode**: Cluster mode enabled
- **Replication**: 2 replicas per shard
- **Persistence**: AOF (Append-Only File)

---

## Rationale

### Why Redis?

**✅ Advantages**:
- **Speed**: In-memory, sub-millisecond latency
- **Rich Data Types**: Strings, hashes, sets, sorted sets, lists
- **TTL Support**: Automatic expiration of keys
- **Atomic Operations**: Thread-safe increments for counters
- **Pub/Sub**: Built-in messaging capabilities
- **Persistence**: RDB + AOF for durability
- **Ecosystem**: Excellent Node.js support (ioredis)
- **Managed Service**: AWS ElastiCache handles ops

**Compared to Alternatives**:

| Feature | Redis | Memcached | DynamoDB DAX | In-Memory JS |
|---------|-------|-----------|--------------|--------------|
| Data Types | ✅ Rich | ❌ Simple | ✅ Rich | ✅ Rich |
| Persistence | ✅ Yes | ❌ No | ✅ Yes | ❌ No |
| Pub/Sub | ✅ Yes | ❌ No | ❌ No | ❌ No |
| TTL | ✅ Yes | ✅ Yes | ✅ Yes | ⚠️ Manual |
| Clustering | ✅ Native | ⚠️ Client-side | ✅ Native | ❌ No |
| Multi-tenancy | ✅ Easy | ⚠️ Limited | ✅ Easy | ❌ No |
| Cost | Medium | Low | High | Free |

---

## Alternatives Considered

### 1. Memcached

**Pros**:
- Simple and fast
- Lower memory overhead
- Multi-threaded

**Cons**:
- ❌ No persistence (data lost on restart)
- ❌ Only key-value pairs (no hashes, sets)
- ❌ No TTL auto-renewal
- ❌ No pub/sub

**Decision**: Rejected - too limited for our needs

---

### 2. DynamoDB with DAX

**Pros**:
- Fully managed
- Native AWS integration
- Microsecond latency

**Cons**:
- ❌ Expensive (~$500/month for our scale)
- ❌ Complex pricing model
- ❌ Overkill for simple caching
- ❌ Vendor lock-in

**Decision**: Rejected - too expensive

---

### 3. In-Memory JavaScript Objects

**Pros**:
- No external dependencies
- Zero latency
- No cost

**Cons**:
- ❌ Lost on pod restart
- ❌ Not shared across pods
- ❌ No persistence
- ❌ Manual eviction logic
- ❌ Memory leaks risk

**Decision**: Rejected - not production-ready

---

## Implementation

### Connection Setup

```javascript
const Redis = require('ioredis');

const redis = new Redis.Cluster([
  {
    host: process.env.REDIS_HOST,
    port: 6379
  }
], {
  redisOptions: {
    password: process.env.REDIS_PASSWORD,
    tls: process.env.NODE_ENV === 'production' ? {} : null
  },
  maxRetriesPerRequest: 3,
  retryDelayOnFailover: 100,
  enableReadyCheck: true,
  scaleReads: 'slave' // Read from replicas
});

redis.on('error', (err) => {
  logger.error('Redis connection error', { error: err.message });
});

redis.on('connect', () => {
  logger.info('Connected to Redis');
});

module.exports = redis;
```

### Cache Strategy

#### 1. Cache-Aside (Lazy Loading)

```javascript
async function getPost(postId) {
  const cacheKey = `post:${postId}`;
  
  // Try cache first
  const cached = await redis.get(cacheKey);
  if (cached) {
    logger.info('Cache hit', { key: cacheKey });
    return JSON.parse(cached);
  }
  
  // Cache miss - fetch from database
  logger.info('Cache miss', { key: cacheKey });
  const post = await db('posts').where('id', postId).first();
  
  if (post) {
    // Store in cache with 1-hour TTL
    await redis.setex(cacheKey, 3600, JSON.stringify(post));
  }
  
  return post;
}
```

#### 2. Write-Through

```javascript
async function updatePost(postId, updates) {
  // Update database
  await db('posts').where('id', postId).update(updates);
  
  // Update cache
  const post = await db('posts').where('id', postId).first();
  await redis.setex(`post:${postId}`, 3600, JSON.stringify(post));
  
  return post;
}
```

#### 3. Cache Invalidation

```javascript
async function deletePost(postId) {
  // Delete from database
  await db('posts').where('id', postId).delete();
  
  // Invalidate cache
  await redis.del(`post:${postId}`);
  await redis.del(`posts:author:${post.authorId}`);
  await redis.del('posts:popular'); // Clear list cache too
}
```

---

## Cache Key Naming

### Convention

Format: `{resource}:{identifier}:{optional-suffix}`

**Examples**:
```
post:123                    # Single post by ID
post:slug:my-post          # Post by slug
posts:author:456           # List of posts by author
user:789:profile           # User profile
tags:popular               # Popular tags list
session:abc123def          # User session
ratelimit:user:456:hour    # Rate limit counter
```

### Benefits

- **Organized**: Easy to find related keys
- **Debuggable**: Clear what each key contains
- **Bulk Operations**: Can delete by pattern (use with caution)

---

## TTL (Time-To-Live) Strategy

| Data Type | TTL | Rationale |
|-----------|-----|-----------|
| Post details | 1 hour | Updated infrequently |
| User profile | 30 minutes | May change (avatar, bio) |
| Popular posts | 5 minutes | Needs to be fresh |
| Search results | 10 minutes | Balance freshness/performance |
| Session data | 24 hours | Match JWT expiry |
| Rate limit counters | 1 hour | Rolling window |
| Password reset tokens | 1 hour | Security + UX |

### Implementation

```javascript
const TTL = {
  POST_DETAIL: 3600,        // 1 hour
  USER_PROFILE: 1800,       // 30 minutes
  POPULAR_POSTS: 300,       // 5 minutes
  SEARCH_RESULTS: 600,      // 10 minutes
  SESSION: 86400,           // 24 hours
  RATE_LIMIT: 3600,         // 1 hour
  TEMP_TOKEN: 3600          // 1 hour
};

await redis.setex(key, TTL.POST_DETAIL, value);
```

---

## Eviction Policy

### Configuration

```
maxmemory 2gb
maxmemory-policy allkeys-lru
```

### Policy Options

| Policy | Behavior | Use Case |
|--------|----------|----------|
| `allkeys-lru` | ✅ Evict least recently used | General caching |
| `allkeys-lfu` | Evict least frequently used | Stable access patterns |
| `volatile-lru` | Evict LRU keys with TTL | Mixed usage (cache + sessions) |
| `noeviction` | Return errors when full | Data that MUST be cached |

**Our Choice**: `allkeys-lru` - balances performance and memory

---

## Monitoring

### Key Metrics

- **Hit Rate**: `hits / (hits + misses)` - Target > 80%
- **Memory Usage**: Current / Max memory - Alert at > 80%
- **Evictions**: Count of evicted keys - Should be low
- **Connections**: Active client connections
- **Commands/sec**: Throughput
- **Latency**: P50, P95, P99

### Dashboard Example

```javascript
// Prometheus metrics
const cacheHits = new promClient.Counter({
  name: 'redis_cache_hits_total',
  help: 'Total cache hits'
});

const cacheMisses = new promClient.Counter({
  name: 'redis_cache_misses_total',
  help: 'Total cache misses'
});

// Track in code
if (cached) {
  cacheHits.inc();
} else {
  cacheMisses.inc();
}
```

### Alerting

```yaml
- alert: LowCacheHitRate
  expr: |
    rate(redis_cache_hits_total[5m]) 
    / (rate(redis_cache_hits_total[5m]) + rate(redis_cache_misses_total[5m])) 
    < 0.8
  annotations:
    summary: Cache hit rate below 80%

- alert: HighMemoryUsage
  expr: redis_memory_used_bytes / redis_memory_max_bytes > 0.8
  annotations:
    summary: Redis memory usage above 80%
```

---

## Deployment

### Infrastructure

- **Provider**: AWS ElastiCache for Redis
- **Instance Type**: cache.r6g.large (2 vCPU, 13.07 GB RAM)
- **Cluster**: 3 shards, 2 replicas each (6 nodes total)
- **Networking**: Private subnet, VPC peering

### Cost Estimate

| Environment | Configuration | Monthly Cost |
|-------------|--------------|--------------|
| Production | 6 x r6g.large | ~$450 |
| Staging | 1 x r6g.medium | ~$80 |
| Development | Local Docker | $0 |
| **Total** | | **~$530/month** |

### Capacity Planning

- **Memory**: 2GB per instance (6GB total across replicas)
- **Throughput**: ~100K ops/sec
- **Connections**: 65K max connections

**Current Usage** (November 2024):
- Memory: 1.2GB (60% utilized)
- Throughput: 15K ops/sec (15% capacity)
- Connections: 50 (0.08% capacity)

**→ Room for 6x growth**

---

## Security

### Encryption

- **In-Transit**: TLS 1.2+ for all connections
- **At-Rest**: EBS volume encryption (AWS managed keys)

### Access Control

- **Auth**: Redis password (32-character random string)
- **Network**: VPC security groups, no public access
- **Encryption**: TLS enforced in production

```javascript
const redis = new Redis({
  host: process.env.REDIS_HOST,
  port: 6379,
  password: process.env.REDIS_PASSWORD,
  tls: process.env.NODE_ENV === 'production' ? {
    rejectUnauthorized: true
  } : null
});
```

---

## Disaster Recovery

### Persistence

**AOF (Append-Only File)** enabled:
- `appendonly yes`
- `appendfsync everysec`

**Snapshots** (RDB):
- Every 5 minutes if ≥ 100 changes
- Daily backup to S3

### Backup Strategy

- **Frequency**: Daily automated snapshots
- **Retention**: 7 days
- **Storage**: S3 encrypted backups
- **RTO**: 15 minutes (restore from snapshot)
- **RPO**: 1 second (AOF)

---

## Migration Path

### Phase 1: Setup (Week 1)
- Provision ElastiCache cluster
- Configure monitoring and alerts
- Test connectivity from services

### Phase 2: Pilot (Week 2)
- Implement caching for `/api/posts` endpoint
- Measure hit rate and latency
- Tune TTLs based on usage

### Phase 3: Rollout (Weeks 3-4)
- Add caching to all read endpoints
- Implement session storage
- Enable rate limiting

### Phase 4: Optimize (Week 5)
- Analyze cache patterns
- Adjust TTLs and eviction policy
- Fine-tune memory allocation

---

## Success Metrics

### Goals (3 months post-deployment)

- ✅ > 80% cache hit rate
- ✅ < 100ms P95 API latency for cached endpoints
- ✅ 70% reduction in database reads
- ✅ Support 1,000 req/sec
- ✅ 99.9% Redis uptime

### Current Status (November 2024)

- ✅ 85% hit rate (exceeds goal)
- ✅ 75ms P95 latency (beats goal)
- ✅ 73% fewer DB reads
- ✅ Handling 800 req/sec (on track)
- ✅ 99.95% uptime

---

## Consequences

### Positive

- ✅ 5x faster API responses for cached data
- ✅ 70% reduction in database load
- ✅ Can handle traffic spikes
- ✅ Improved user experience
- ✅ Lower database costs (smaller instance)

### Negative

- ❌ Added complexity (cache invalidation)
- ❌ Additional cost (~$530/month)
- ❌ Potential stale data
- ❌ Need to monitor cache health

### Neutral

- Data consistency is eventually consistent
- Team needs Redis expertise

---

## Best Practices

### DO ✅

- Set appropriate TTLs for each data type
- Implement cache invalidation on updates
- Monitor hit rates and adjust strategy
- Use consistent key naming conventions
- Handle cache failures gracefully
- Log cache hits/misses for debugging

### DON'T ❌

- Cache everything (be selective)
- Use very long TTLs for frequently changing data
- Ignore cache failures (have fallback)
- Store large objects (>1MB) in cache
- Use cache as primary data store
- Delete keys by pattern in production (slow)

---

## Related Documentation

- [Data Protection](../security/data-protection.md)
- [API Security](../security/api-security.md)
- [ADR-002: PostgreSQL](adr-002-postgresql.md)

---

## References

- [Redis Official Documentation](https://redis.io/documentation)
- [AWS ElastiCache Best Practices](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/BestPractices.html)
- [Caching Strategies](https://aws.amazon.com/caching/best-practices/)

---

**Approved By**: CTO (Emily Zhang), VP Engineering (Robert Wilson)  
**Implementation Lead**: Backend Team B  
**Review Date**: October 20, 2025 (1 year)
