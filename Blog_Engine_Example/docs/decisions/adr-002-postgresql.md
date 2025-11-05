# ADR-002: PostgreSQL as Primary Database

**Status**: ✅ Accepted  
**Date**: November 2, 2024  
**Deciders**: Architecture Team, Backend Team  
**Related**: ADR-001 (Microservices), Database Schema  
**Supersedes**: -

---

## Context

Need to select a primary database for Blog Engine services. Requirements:

1. **ACID transactions**: Blog posts, comments need strong consistency
2. **Relational data**: Users, posts, comments have clear relationships
3. **Full-text search**: Need to search posts by title/content
4. **JSON support**: User preferences, metadata stored as JSON
5. **Scalability**: Handle 100K+ posts, 1M+ comments
6. **Read replicas**: Scale read-heavy workloads (posts, user profiles)
7. **Mature ecosystem**: Well-supported, good tooling
8. **Cost**: Must fit startup budget (~$500/month for database)

---

## Decision

Use **PostgreSQL 14+** as the primary database for all microservices.

**Deployment**:
- AWS RDS PostgreSQL Multi-AZ for production
- Single instance for staging/dev
- Automated daily backups, 7-day retention
- Point-in-time recovery enabled

**Configuration**:
- Each service has its own database schema (logical isolation)
- Connection pooling (20-100 connections per service)
- Read replicas for high-traffic services (Post, User)

---

## Consequences

### Positive

1. **ACID compliance**: Strong consistency for critical data
2. **Mature & stable**: 30+ years of development, production-proven
3. **Full-text search**: Built-in support (tsvector), no separate search engine initially
4. **JSON support**: JSONB type for flexible schema parts
5. **Great tooling**: pg Admin, migration tools (Flyway), ORMs (Sequelize, TypeORM)
6. **Read replicas**: Built-in support for scaling reads
7. **Cost-effective**: RDS pricing ~$200-400/month for our scale
8. **Team familiarity**: Team already knows PostgreSQL well

### Negative

1. **Not horizontally scalable**: Sharding is complex (acceptable for our scale)
2. **Read replica lag**: Eventual consistency for replicas (5-10sec typical)
3. **Vendor management**: RDS adds dependency, though multi-cloud possible
4. **Connection limits**: Need connection pooling (mitigated with PgBouncer if needed)

### Neutral

- Need to manage migrations carefully (Flyway/Liquibase)
- Monitoring needed for slow queries (pg_stat_statements)

---

## Alternatives Considered

### Alternative 1: MongoDB
**Description**: Document database, schema-less  
**Pros**: Flexible schema, horizontal scaling, good for rapid prototyping  
**Cons**: Eventual consistency challenges, no JOIN support, weaker ACID guarantees  
**Why Not**: Blog engine has clear relational structure. Strong consistency needed for posts/comments.

### Alternative 2: MySQL
**Description**: Popular open-source RDBMS  
**Pros**: Very popular, great performance, AWS RDS support  
**Cons**: Weaker JSON support, less advanced features, more complex replication setup  
**Why Not**: PostgreSQL JSON support superior. Team prefers PostgreSQL's feature set.

### Alternative 3: DynamoDB
**Description**: AWS-managed NoSQL database  
**Pros**: Fully managed, auto-scaling, serverless  
**Cons**: No JOIN support, complex queries difficult, vendor lock-in, unpredictable costs  
**Why Not**: Relational queries essential. Want to avoid AWS lock-in for database.

---

## Implementation

**Affected services**: All (Auth, Post, Comment, User, Notification, Admin)

**Database per service**:
```
blogengine_prod.auth_schema      # Auth Service
blogengine_prod.post_schema      # Post Service
blogengine_prod.comment_schema   # Comment Service
blogengine_prod.user_schema      # User Service
blogengine_prod.notification_schema # Notification Service
blogengine_prod.admin_schema     # Admin Service
```

**Shared infrastructure**:
- Single RDS instance, separate schemas (cost optimization)
- Can split to separate instances if needed later

**Migration strategy**:
- Flyway for schema migrations (decision pending in ADR-006)
- One migration folder per service

---

## References

**Implementation**:
- [Database Schema](../../reference/database/schema.md)
- [Configuration](../../reference/configuration/parameters.md#database-configuration)

**Related ADRs**:
- [ADR-001: Microservices](adr-001-microservices.md)
- [ADR-006: Migration Tool](adr-006-migration-tool.md) (pending)

**External**:
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [AWS RDS PostgreSQL](https://aws.amazon.com/rds/postgresql/)

---

## Notes

**Watch for**:
- Connection pool exhaustion (monitor active connections)
- Slow queries (set up pg_stat_statements)
- Storage growth (100GB initially, can expand)

**Reconsider if**:
- Data grows beyond 10TB (consider sharding/Citus)
- Need sub-10ms query latency globally (consider caching layer)
- Budget for database exceeds $1K/month

**Success metrics**:
- Query response time: P95 < 50ms
- Database CPU: < 70% average
- Connection pool: < 80% utilization

**Review schedule**: Quarterly performance review

---

**Author**: Backend Team  
**Reviewers**: Architecture Team, DevOps  
**Last Updated**: November 2, 2024
