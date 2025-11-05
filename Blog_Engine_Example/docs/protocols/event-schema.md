# Event Schema Standards

> **Document Type**: Protocol  
> **Audience**: All service developers  
> **Last Updated**: November 5, 2024  
> **Status**: ✅ Approved & Enforced

---

## Overview

This document defines the standard event schema for all asynchronous events in the Blog Engine system using RabbitMQ as the message broker.

**All services MUST publish and consume events using this schema.**

---

## Event Structure

### Standard Envelope

```json
{
  "eventId": "evt_abc123def456",
  "eventType": "post.created",
  "eventVersion": "1.0",
  "timestamp": "2024-11-05T14:30:00Z",
  "source": {
    "service": "post-service",
    "version": "2.1.0",
    "instance": "post-service-pod-123"
  },
  "correlation": {
    "requestId": "req_xyz789",
    "userId": 123,
    "sessionId": "sess_abc"
  },
  "payload": {
    // Event-specific data
  },
  "metadata": {
    "retryCount": 0,
    "priority": "normal"
  }
}
```

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `eventId` | string | Unique event identifier (UUID v4) |
| `eventType` | string | Event type in `resource.action` format |
| `eventVersion` | string | Schema version (semver) |
| `timestamp` | string | ISO 8601 timestamp (UTC) |
| `source` | object | Event source information |
| `correlation` | object | Request correlation data |
| `payload` | object | Event-specific data |

### Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `metadata` | object | Additional metadata (retries, priority) |

---

## Event Types

### Naming Convention

Format: `{resource}.{action}`

**Examples**:
- `post.created`
- `post.updated`
- `post.deleted`
- `user.registered`
- `comment.flagged`

### Categories

#### Post Events

| Event Type | Description | Payload |
|------------|-------------|---------|
| `post.created` | New post created | `{postId, authorId, title, status}` |
| `post.updated` | Post updated | `{postId, authorId, changes}` |
| `post.published` | Post published | `{postId, authorId, publishedAt}` |
| `post.deleted` | Post deleted | `{postId, authorId, deletedAt}` |
| `post.liked` | Post liked by user | `{postId, userId}` |
| `post.viewed` | Post viewed | `{postId, userId, viewCount}` |

#### User Events

| Event Type | Description | Payload |
|------------|-------------|---------|
| `user.registered` | New user signed up | `{userId, email, registeredAt}` |
| `user.updated` | User profile updated | `{userId, changes}` |
| `user.deleted` | User account deleted | `{userId, deletedAt}` |
| `user.login` | User logged in | `{userId, ipAddress, timestamp}` |
| `user.logout` | User logged out | `{userId, timestamp}` |

#### Comment Events

| Event Type | Description | Payload |
|------------|-------------|---------|
| `comment.created` | New comment posted | `{commentId, postId, userId, content}` |
| `comment.updated` | Comment edited | `{commentId, changes}` |
| `comment.deleted` | Comment removed | `{commentId, deletedBy}` |
| `comment.flagged` | Comment reported | `{commentId, flaggedBy, reason}` |

#### Notification Events

| Event Type | Description | Payload |
|------------|-------------|---------|
| `notification.sent` | Notification delivered | `{notificationId, userId, channel}` |
| `notification.failed` | Notification failed | `{notificationId, error}` |

---

## Event Examples

### post.created

```json
{
  "eventId": "evt_550e8400-e29b-41d4-a716-446655440000",
  "eventType": "post.created",
  "eventVersion": "1.0",
  "timestamp": "2024-11-05T14:30:00Z",
  "source": {
    "service": "post-service",
    "version": "2.1.0",
    "instance": "post-service-pod-abc123"
  },
  "correlation": {
    "requestId": "req_abc123def456",
    "userId": 123,
    "sessionId": "sess_xyz789"
  },
  "payload": {
    "postId": 157,
    "authorId": 123,
    "title": "Getting Started with Node.js",
    "slug": "getting-started-with-nodejs",
    "status": "draft",
    "createdAt": "2024-11-05T14:30:00Z"
  },
  "metadata": {
    "retryCount": 0,
    "priority": "normal"
  }
}
```

### post.published

```json
{
  "eventId": "evt_7f3e8bd0-c7a2-4d9f-9b1e-5c8a3f2d1e0b",
  "eventType": "post.published",
  "eventVersion": "1.0",
  "timestamp": "2024-11-05T15:00:00Z",
  "source": {
    "service": "post-service",
    "version": "2.1.0",
    "instance": "post-service-pod-abc123"
  },
  "correlation": {
    "requestId": "req_def456ghi789",
    "userId": 123
  },
  "payload": {
    "postId": 157,
    "authorId": 123,
    "title": "Getting Started with Node.js",
    "slug": "getting-started-with-nodejs",
    "publishedAt": "2024-11-05T15:00:00Z",
    "subscriberCount": 42
  },
  "metadata": {
    "retryCount": 0,
    "priority": "high"
  }
}
```

### user.registered

```json
{
  "eventId": "evt_9a2b3c4d-5e6f-7g8h-9i0j-1k2l3m4n5o6p",
  "eventType": "user.registered",
  "eventVersion": "1.0",
  "timestamp": "2024-11-05T10:00:00Z",
  "source": {
    "service": "auth-service",
    "version": "1.5.0",
    "instance": "auth-service-pod-xyz789"
  },
  "correlation": {
    "requestId": "req_user_signup_123",
    "sessionId": "sess_new_user"
  },
  "payload": {
    "userId": 456,
    "email": "newuser@example.com",
    "name": "Jane Doe",
    "registeredAt": "2024-11-05T10:00:00Z",
    "emailVerified": false
  },
  "metadata": {
    "retryCount": 0,
    "priority": "normal"
  }
}
```

### comment.flagged

```json
{
  "eventId": "evt_a1b2c3d4-e5f6-g7h8-i9j0-k1l2m3n4o5p6",
  "eventType": "comment.flagged",
  "eventVersion": "1.0",
  "timestamp": "2024-11-05T16:45:00Z",
  "source": {
    "service": "comment-service",
    "version": "1.3.0",
    "instance": "comment-service-pod-def456"
  },
  "correlation": {
    "requestId": "req_flag_comment",
    "userId": 789
  },
  "payload": {
    "commentId": 1234,
    "postId": 157,
    "flaggedBy": 789,
    "reason": "spam",
    "flaggedAt": "2024-11-05T16:45:00Z",
    "content": "Comment text here...",
    "authorId": 456
  },
  "metadata": {
    "retryCount": 0,
    "priority": "high"
  }
}
```

---

## RabbitMQ Configuration

### Exchange Types

**Topic Exchange**: `blog-engine.events`

All events are published to this exchange with routing keys based on event type.

### Routing Keys

Format: `{resource}.{action}.{environment}`

**Examples**:
- `post.created.production`
- `user.registered.staging`
- `comment.flagged.production`

### Queue Naming

Format: `{service-name}.{event-type}`

**Examples**:
- `notification-service.post.published`
- `analytics-service.user.registered`
- `admin-service.comment.flagged`

### Queue Bindings

```javascript
// Notification service subscribes to post.published
channel.bindQueue(
  'notification-service.post.published',
  'blog-engine.events',
  'post.published.#'
);

// Analytics service subscribes to all user events
channel.bindQueue(
  'analytics-service.user.*',
  'blog-engine.events',
  'user.*.#'
);
```

---

## Publishing Events

### Publisher Implementation

```javascript
const amqp = require('amqplib');
const { v4: uuidv4 } = require('uuid');

class EventPublisher {
  constructor(connection) {
    this.connection = connection;
    this.channel = null;
  }

  async init() {
    this.channel = await this.connection.createChannel();
    await this.channel.assertExchange('blog-engine.events', 'topic', {
      durable: true
    });
  }

  async publish(eventType, payload, correlation = {}) {
    const event = {
      eventId: uuidv4(),
      eventType,
      eventVersion: '1.0',
      timestamp: new Date().toISOString(),
      source: {
        service: process.env.SERVICE_NAME,
        version: process.env.SERVICE_VERSION,
        instance: process.env.HOSTNAME
      },
      correlation: {
        requestId: correlation.requestId,
        userId: correlation.userId,
        sessionId: correlation.sessionId
      },
      payload,
      metadata: {
        retryCount: 0,
        priority: correlation.priority || 'normal'
      }
    };

    const routingKey = `${eventType}.${process.env.NODE_ENV}`;

    await this.channel.publish(
      'blog-engine.events',
      routingKey,
      Buffer.from(JSON.stringify(event)),
      {
        persistent: true,
        contentType: 'application/json',
        messageId: event.eventId,
        timestamp: Date.now()
      }
    );

    logger.info('Event published', {
      eventId: event.eventId,
      eventType,
      routingKey
    });

    return event;
  }
}

// Usage
await eventPublisher.publish('post.created', {
  postId: 157,
  authorId: 123,
  title: 'My Post',
  status: 'draft'
}, {
  requestId: req.id,
  userId: req.user.id
});
```

---

## Consuming Events

### Consumer Implementation

```javascript
class EventConsumer {
  constructor(connection, queueName, eventHandlers) {
    this.connection = connection;
    this.queueName = queueName;
    this.eventHandlers = eventHandlers;
    this.channel = null;
  }

  async init() {
    this.channel = await this.connection.createChannel();
    
    // Assert queue
    await this.channel.assertQueue(this.queueName, {
      durable: true
    });

    // Prefetch: process one message at a time
    await this.channel.prefetch(1);
  }

  async consume() {
    await this.channel.consume(this.queueName, async (msg) => {
      if (!msg) return;

      try {
        const event = JSON.parse(msg.content.toString());
        
        logger.info('Event received', {
          eventId: event.eventId,
          eventType: event.eventType
        });

        // Get handler for event type
        const handler = this.eventHandlers[event.eventType];
        
        if (!handler) {
          logger.warn('No handler for event type', {
            eventType: event.eventType
          });
          this.channel.ack(msg);
          return;
        }

        // Handle event
        await handler(event);

        // Acknowledge message
        this.channel.ack(msg);

        logger.info('Event processed successfully', {
          eventId: event.eventId
        });

      } catch (error) {
        logger.error('Event processing failed', {
          error: error.message,
          stack: error.stack
        });

        // Reject and requeue (with limit)
        const retryCount = event.metadata?.retryCount || 0;
        
        if (retryCount < 3) {
          this.channel.nack(msg, false, true); // Requeue
        } else {
          this.channel.nack(msg, false, false); // Dead letter
          logger.error('Event sent to dead letter queue', {
            eventId: event.eventId,
            retryCount
          });
        }
      }
    });
  }
}

// Usage
const consumer = new EventConsumer(
  connection,
  'notification-service.post.published',
  {
    'post.published': async (event) => {
      await notificationService.sendPostPublishedNotification(event.payload);
    }
  }
);

await consumer.init();
await consumer.consume();
```

---

## Event Versioning

### Version Strategy

Use semantic versioning: `MAJOR.MINOR.PATCH`

- **MAJOR**: Breaking changes (remove fields, change types)
- **MINOR**: New fields added (backward compatible)
- **PATCH**: Bug fixes, clarifications

### Version Field

```json
{
  "eventVersion": "2.1.0",
  ...
}
```

### Handling Version Changes

```javascript
async function handlePostCreated(event) {
  const version = event.eventVersion.split('.')[0]; // Get major version

  switch (version) {
    case '1':
      return handlePostCreatedV1(event);
    case '2':
      return handlePostCreatedV2(event);
    default:
      throw new Error(`Unsupported event version: ${event.eventVersion}`);
  }
}
```

---

## Error Handling

### Dead Letter Queue

Configure DLQ for failed messages:

```javascript
await channel.assertQueue('blog-engine.events.dlq', {
  durable: true
});

await channel.assertQueue('notification-service.post.published', {
  durable: true,
  arguments: {
    'x-dead-letter-exchange': 'blog-engine.events.dlq.exchange',
    'x-dead-letter-routing-key': 'notification-service.post.published.failed'
  }
});
```

### Retry Strategy

1. **Immediate retry**: 3 attempts with exponential backoff
2. **Delayed retry**: After 1 hour
3. **Dead letter**: After all retries exhausted
4. **Manual intervention**: Inspect DLQ and reprocess

---

## Idempotency

### Duplicate Detection

Consumers MUST handle duplicate events using `eventId`:

```javascript
async function handleEvent(event) {
  // Check if already processed
  const processed = await redis.get(`event:${event.eventId}`);
  
  if (processed) {
    logger.info('Event already processed', { eventId: event.eventId });
    return;
  }

  // Process event
  await processEvent(event);

  // Mark as processed (TTL: 7 days)
  await redis.setex(`event:${event.eventId}`, 604800, 'processed');
}
```

---

## Monitoring

### Metrics to Track

- Events published per type
- Events consumed per queue
- Processing latency
- Retry count distribution
- Dead letter queue size

### Prometheus Metrics

```javascript
const eventPublished = new promClient.Counter({
  name: 'events_published_total',
  help: 'Total events published',
  labelNames: ['event_type', 'service']
});

const eventProcessingDuration = new promClient.Histogram({
  name: 'event_processing_duration_seconds',
  help: 'Event processing duration',
  labelNames: ['event_type', 'status']
});
```

---

## Best Practices

### DO ✅

- **Use unique eventId** for every event (UUID v4)
- **Include correlation data** for request tracing
- **Keep payloads small** (< 10KB)
- **Version your events** from the start
- **Handle duplicates** idempotently
- **Log all events** published and consumed
- **Monitor queue depths** and processing rates

### DON'T ❌

- **Include sensitive data** in events (passwords, tokens)
- **Make events too large** (>100KB)
- **Change event schema** without versioning
- **Assume order** of event delivery
- **Ignore failed events** in DLQ
- **Block event handlers** with synchronous operations

---

## Related Documentation

- [Error Handling Standards](error-handling.md)
- [Logging Standards](logging-standards.md)
- [ADR-004: RabbitMQ for Event Bus](../decisions/adr-004-rabbitmq.md)

---

**Maintained By**: Architecture Team  
**Review Frequency**: Quarterly  
**Next Review**: February 1, 2025
