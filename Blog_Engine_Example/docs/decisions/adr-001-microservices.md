# ADR-001: Microservices Architecture

**Status**: ✅ Accepted  
**Date**: November 1, 2024  
**Deciders**: Architecture Team, Engineering Leadership  
**Related**: All features, System Architecture  
**Supersedes**: -

---

## Context

When planning the Blog Engine, we faced several challenges:

1. **Scaling concerns**: Different parts have different load patterns
   - Posts: High read, lower write (est. 1000 req/sec)
   - Comments: Moderate read/write, spiky (est. 100 req/sec)
   - Notifications: Async, batch processing (bursty)
   - Admin: Low traffic, different SLA

2. **Team structure**: 8 developers initially, growing to 15+ in year 2
   - Need independent deployment cadence per team
   - Want to minimize merge conflicts
   - Clear ownership boundaries required

3. **Technology flexibility**: Different services benefit from different tech
   - Real-time features may need WebSocket optimization
   - Image processing may benefit from Go/Python
   - API services work well in Node.js

4. **Failure isolation**: Don't want one bug taking down entire system

5. **Timeline**: Initial MVP in 8-9 weeks, long-term maintainability critical

---

## Decision

We will implement a **microservices architecture** with the following core services:

- **API Gateway** (port 3000): Single entry point, routing, auth validation
- **Auth Service** (port 3001): Authentication, JWT, OAuth2
- **Post Service** (port 3002): Blog posts CRUD, S3 integration
- **Comment Service** (port 3003): Comments with moderation
- **User Service** (port 3004): User profiles, preferences
- **Notification Service** (port 3005): Email + push notifications
- **Admin Service** (port 3006): Admin panel, moderation queue

**Key principles**:
- Each service owns its data (separate DB schemas)
- Services communicate via REST (synchronous) or RabbitMQ (async events)
- Independent deployment and scaling
- Kubernetes orchestration
- Centralized logging and monitoring

---

## Consequences

### Positive

1. **Independent scaling**: Scale services based on load (30-40% cost savings vs scaling everything)
2. **Team autonomy**: Clear ownership, parallel development, faster velocity after Month 6
3. **Technology flexibility**: Use best tool per service (Node.js primary, Python/Go for specific needs)
4. **Failure isolation**: Service crashes don't take down entire system (99.9%+ availability target)
5. **Deployment independence**: Deploy services separately, reducing risk and testing scope
6. **Faster long-term velocity**: 40% faster feature delivery after Month 7

### Negative

1. **Operational complexity**: More to monitor, deploy, and debug
   - *Mitigation*: Centralized logging (ELK), tracing (Jaeger), Kubernetes orchestration

2. **Distributed system challenges**: Network failures, latency, timeouts
   - *Mitigation*: Circuit breakers, retry logic, fallback mechanisms

3. **Data consistency**: No distributed transactions, eventual consistency
   - *Mitigation*: Event-driven architecture with RabbitMQ, compensating transactions

4. **Initial development time**: 50% longer MVP (8-9 weeks vs 6 weeks for monolith)
   - *Mitigation*: Accept upfront cost, break-even at 6 months

5. **Testing complexity**: Need integration and contract tests
   - *Mitigation*: Layered testing strategy (unit → contract → integration → E2E)

### Neutral

- Requires DevOps expertise (2-3 month investment in infrastructure)
- Steeper learning curve for new developers
- More configuration management needed

---

## Alternatives Considered

### Alternative 1: Monolithic Architecture
**Description**: Single deployable application with all features  
**Pros**: Simpler deployment, easier debugging, faster initial development  
**Cons**: Can't scale parts independently, bottleneck with 8+ developers, harder to enforce boundaries  
**Why Not**: Team size (8 → 15) would create merge conflicts and slow velocity. Different scaling needs per feature.

### Alternative 2: Modular Monolith
**Description**: Single deployable with well-defined module boundaries  
**Pros**: Module boundaries without microservices complexity, easier than full microservices  
**Cons**: Can't scale modules independently, still deploy everything together, database becomes bottleneck  
**Why Not**: Need independent scaling and deployment. Code drift makes boundaries hard to enforce.

### Alternative 3: Serverless Functions (AWS Lambda)
**Description**: Function-per-endpoint, fully managed infrastructure  
**Pros**: No infrastructure management, auto-scaling, pay-per-use  
**Cons**: Cold start latency (P99 ~500ms, need <200ms), vendor lock-in, unclear cost at scale, complex debugging  
**Why Not**: Latency requirements and vendor lock-in concerns. Cost unpredictable at target scale (10K+ users).

---

## Implementation

**Phase 1** (Weeks 1-3): Infrastructure
- Set up Kubernetes cluster (AWS EKS)
- Configure CI/CD pipeline (GitHub Actions)
- Set up logging (CloudWatch), monitoring (Prometheus)
- Create service templates

**Phase 2** (Weeks 4-6): Core Services
- Implement Auth Service
- Implement API Gateway
- Set up RabbitMQ message bus
- Implement Post Service

**Phase 3** (Weeks 7-9): Additional Services
- Implement Comment, User, Notification services
- Integration testing
- Performance testing
- MVP launch

**Services affected**: All  
**Configuration**: See `/docs/architecture/system-overview.md` and `/reference/configuration/parameters.md`  
**Migration**: N/A (greenfield project)

---

## References

**Architecture**:
- [System Architecture Overview](../architecture/system-overview.md)
- [Service Catalog](../architecture/services/README.md)

**Implementation**:
- [Configuration Parameters](../../reference/configuration/parameters.md)
- [Deployment Guide](../../runbooks/deployment.md)

**Related ADRs**:
- [ADR-002: PostgreSQL](adr-002-postgresql.md) - Database choice
- [ADR-004: RabbitMQ](adr-004-rabbitmq.md) - Event bus

**Features using this**:
- All features in `/features/`

**External resources**:
- [Martin Fowler - Microservices](https://martinfowler.com/articles/microservices.html)
- [Kubernetes Best Practices](https://kubernetes.io/docs/concepts/)

---

## Notes

**Watch for**:
- Over-fragmentation: Don't create services too small (min 2-3 resources/tables per service)
- Performance bottlenecks: Monitor inter-service call latency
- Distributed debugging: Ensure correlation IDs across all logs

**Reconsider if**:
- Team shrinks to < 4 developers
- System complexity doesn't grow as expected
- Operational costs exceed 20% of total budget

**Success metrics**:
- Deployment frequency: Target 2+ deploys/day per service by Month 6
- Mean time to recovery: < 15 minutes
- Service availability: 99.9%+ per service
- Developer velocity: 40% faster feature delivery by Month 7

**Review schedule**: Quarterly architecture review

---

**Author**: Architecture Team  
**Reviewers**: CTO, Engineering Manager, DevOps Lead  
**Last Updated**: November 1, 2024
