# ADR-004: RabbitMQ for Event Bus

**Status**: ✅ Accepted  
**Date**: October 15, 2024  
**Deciders**: Architecture Team (Sarah Johnson, Mike Chen, David Kim)  
**Last Updated**: November 5, 2024

---

## Context

The Blog Engine system needs an event-driven architecture to enable loose coupling between microservices and support asynchronous communication patterns.

### Requirements

- **Reliable message delivery**: Guarantee delivery of critical events
- **Pub/Sub pattern**: Multiple consumers can subscribe to same events
- **Message ordering**: Preserve order where needed
- **Scalability**: Handle 10K+ events per second
- **Durability**: Persist messages to survive crashes
- **Dead letter handling**: Handle failed message processing
- **Monitoring**: Track message flow and health

### Use Cases

- **Post published**: Notify subscribers, update search index, analytics
- **User registered**: Send welcome email, create profile, track metrics
- **Comment flagged**: Alert moderators, increment spam score
- **Payment processed**: Update subscription, send receipt, provision access

---

## Decision

We will use **RabbitMQ** as our message broker for the event bus.

### Architecture

```
┌────────────┐                           ┌────────────┐
│   Post     │ --publish--> [Exchange] --│ Notification│
│  Service   │              (Topic)    └>│  Service    │
└────────────┘                           └────────────┘
                                              ↓
                                         [Queue]
```

### Key Components

1. **Topic Exchange**: `blog-engine.events`
2. **Routing Keys**: `{resource}.{action}.{environment}`
3. **Queues**: Per-service, per-event-type
4. **Dead Letter Queue**: For failed messages

---

## Rationale

### Why RabbitMQ?

**✅ Advantages**:
- **Mature & Stable**: 15+ years in production use
- **AMQP Protocol**: Industry standard, well-documented
- **Management UI**: Built-in monitoring and admin interface
- **Flexible Routing**: Topic exchanges support complex routing
- **Message Persistence**: Durable queues and messages
- **Community Support**: Large ecosystem, extensive documentation
- **Client Libraries**: Excellent Node.js support (amqplib)

**Compared to Alternatives**:

| Feature | RabbitMQ | Kafka | AWS SQS | Redis Pub/Sub |
|---------|----------|-------|---------|---------------|
| Message Ordering | ✅ Per queue | ✅ Per partition | ❌ No | ❌ No |
| Message Persistence | ✅ Yes | ✅ Yes | ✅ Yes | ❌ No |
| Pub/Sub | ✅ Yes | ✅ Yes | ❌ No | ✅ Yes |
| Dead Letter Queue | ✅ Yes | ❌ Complex | ✅ Yes | ❌ No |
| Operational Complexity | Medium | High | Low | Low |
| Cost (AWS) | EC2 + EBS | MSK (expensive) | Per message | ElastiCache |
| Learning Curve | Medium | Steep | Easy | Easy |

---

## Alternatives Considered

### 1. Apache Kafka

**Pros**:
- Exceptional throughput (millions msg/sec)
- Built for log aggregation and streaming
- Replay capability

**Cons**:
- ❌ Overkill for our scale (10K msg/sec)
- ❌ Complex to operate
- ❌ Steep learning curve
- ❌ AWS MSK is expensive ($200+/month)

**Decision**: Rejected - unnecessary complexity for our needs

---

### 2. AWS SQS/SNS

**Pros**:
- Fully managed (no ops)
- Auto-scaling
- Simple to use

**Cons**:
- ❌ No message ordering within topic
- ❌ Limited routing flexibility
- ❌ Cost adds up with volume
- ❌ Vendor lock-in
- ❌ No local development (requires AWS credentials)

**Decision**: Rejected - too limiting, vendor lock-in

---

### 3. Redis Pub/Sub

**Pros**:
- Simple and fast
- Already using Redis for caching

**Cons**:
- ❌ No message persistence
- ❌ No delivery guarantees
- ❌ No message replay
- ❌ No dead letter handling

**Decision**: Rejected - insufficient reliability

---

## Implementation

### Connection Setup

```javascript
const amqp = require('amqplib');

async function connectRabbitMQ() {
  const connection = await amqp.connect({
    protocol: 'amqp',
    hostname: process.env.RABBITMQ_HOST,
    port: 5672,
    username: process.env.RABBITMQ_USER,
    password: process.env.RABBITMQ_PASSWORD,
    vhost: '/',
    heartbeat: 60
  });
  
  connection.on('error', (err) => {
    logger.error('RabbitMQ connection error', { error: err.message });
  });
  
  connection.on('close', () => {
    logger.warn('RabbitMQ connection closed, reconnecting...');
    setTimeout(connectRabbitMQ, 5000);
  });
  
  return connection;
}
```

### Exchange & Queue Configuration

```javascript
// Create exchange
await channel.assertExchange('blog-engine.events', 'topic', {
  durable: true
});

// Create queue with DLQ
await channel.assertQueue('notification-service.post.published', {
  durable: true,
  arguments: {
    'x-dead-letter-exchange': 'blog-engine.dlq',
    'x-dead-letter-routing-key': 'notification-service.failed',
    'x-message-ttl': 86400000 // 24 hours
  }
});

// Bind queue to exchange
await channel.bindQueue(
  'notification-service.post.published',
  'blog-engine.events',
  'post.published.#'
);
```

### Publishing Events

```javascript
await channel.publish(
  'blog-engine.events',
  'post.published.production',
  Buffer.from(JSON.stringify(event)),
  {
    persistent: true,
    contentType: 'application/json',
    messageId: event.eventId
  }
);
```

---

## Deployment

### Infrastructure

- **Provider**: AWS EC2 (t3.medium instances)
- **Cluster**: 3-node cluster for high availability
- **Disk**: EBS GP3 volumes for message persistence
- **Networking**: Private subnet with NAT gateway
- **Load Balancer**: ALB for management UI

### Resource Requirements

- **Production**: 3 x t3.medium (2 vCPU, 4GB RAM)
- **Staging**: 1 x t3.small (2 vCPU, 2GB RAM)
- **Development**: Local Docker container

### Cost Estimate

| Environment | Monthly Cost |
|-------------|--------------|
| Production | ~$150 (3 EC2 + EBS) |
| Staging | ~$25 (1 EC2 + EBS) |
| Total | ~$175/month |

**vs Kafka on AWS MSK**: ~$600/month (4x more expensive)

---

## Monitoring

### Metrics to Track

- Queue depth (messages waiting)
- Message rate (published/consumed per second)
- Consumer lag
- Error rate
- Connection count
- Dead letter queue size

### Alerting

```yaml
- alert: HighQueueDepth
  expr: rabbitmq_queue_messages > 10000
  for: 5m
  annotations:
    summary: Queue has too many messages

- alert: DeadLetterQueueGrowing
  expr: rate(rabbitmq_queue_messages{queue=~".*dlq"}[5m]) > 0
  annotations:
    summary: Messages failing to process
```

---

## Migration Path

### Phase 1: Setup (Week 1)
- Deploy RabbitMQ cluster
- Configure exchanges and queues
- Set up monitoring

### Phase 2: Pilot (Week 2)
- Migrate `post.published` event
- Test with notification service
- Validate reliability

### Phase 3: Rollout (Weeks 3-4)
- Migrate all events
- Decomm custom event system
- Monitor and optimize

---

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Message loss | Low | High | Durable queues, publisher confirms |
| Broker downtime | Medium | High | 3-node cluster, auto-recovery |
| Queue buildup | Medium | Medium | Consumer auto-scaling, DLQ |
| Network partition | Low | High | Heartbeats, connection monitoring |
| Operational complexity | Medium | Medium | Managed via Helm charts, runbooks |

---

## Success Metrics

### Goals (3 months post-deployment)

- ✅ 99.9% message delivery rate
- ✅ < 500ms average latency (publish to consume)
- ✅ Zero data loss incidents
- ✅ < 1% messages in DLQ
- ✅ 100% of events migrated

### Current Status (November 2024)

- ✅ 99.95% delivery rate (exceeds goal)
- ✅ 245ms average latency
- ✅ Zero data loss
- ✅ 0.3% DLQ rate
- ✅ All critical events migrated

---

## Consequences

### Positive

- ✅ Microservices are loosely coupled
- ✅ Asynchronous processing reduces API latency
- ✅ Easy to add new consumers
- ✅ Message persistence ensures reliability
- ✅ Dead letter queues catch failures

### Negative

- ❌ Added operational complexity
- ❌ Need to monitor another component
- ❌ Learning curve for team
- ❌ Eventual consistency (not immediate)

### Neutral

- Infrastructure cost (~$175/month)
- Need RabbitMQ expertise on team

---

## Related Documentation

- [Event Schema Standards](../protocols/event-schema.md)
- [Logging Standards](../protocols/logging-standards.md)
- [ADR-001: Microservices Architecture](adr-001-microservices.md)

---

## References

- [RabbitMQ Official Documentation](https://www.rabbitmq.com/documentation.html)
- [AMQP Protocol Specification](https://www.amqp.org/)
- [RabbitMQ on AWS Best Practices](https://aws.amazon.com/blogs/compute/introducing-rabbitmq-on-amazon-mq/)

---

**Approved By**: CTO (Emily Zhang), VP Engineering (Robert Wilson)  
**Implementation Lead**: Backend Team A  
**Review Date**: October 15, 2025 (1 year)
