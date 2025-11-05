# Message Queue Configuration

> **Document Type**: Reference (Information-Oriented)  
> **Category**: Configuration  
> **Version**: 1.0  
> **Last Updated**: 01/20/2025

## Overview

Configuration reference for the message broker used for event publishing and background processing.

**Quick Facts**:
- **Technology**: RabbitMQ
- **Default Exchange**: `domain.events`
- **Protocol**: AMQP 0-9-1
- **Status**: Stable

---

## Synopsis / Quick Reference

Example environment variables:

```bash
RABBITMQ_URL=amqp://user:password@rabbitmq:5672/
EVENT_EXCHANGE=domain.events
EVENT_EXCHANGE_TYPE=topic
EVENT_PUBLISHER_CONFIRM=true
```

---

## Description

This document defines the configuration parameters for connecting to and using the message broker. It applies to all services that publish or consume events.

### Key Characteristics
- Durable exchanges and queues for reliability
- Topic-based routing for domain events
- Publisher confirms for delivery assurance

---

## Parameters

### Connection

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `RABBITMQ_URL` | string | Yes | - | AMQP connection URL including credentials |
| `RABBITMQ_HEARTBEAT` | integer | No | 30 | Heartbeat interval in seconds |
| `RABBITMQ_PREFETCH` | integer | No | 10 | Max unacknowledged messages per consumer |

### Exchanges

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `EVENT_EXCHANGE` | string | No | `domain.events` | Exchange for domain events |
| `EVENT_EXCHANGE_TYPE` | string | No | `topic` | Exchange type (`direct`, `topic`, `fanout`) |

### Publishing

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `EVENT_PUBLISHER_CONFIRM` | boolean | No | true | Use publisher confirms for reliability |
| `EVENT_DELIVERY_MODE` | integer | No | 2 | 2 = persistent messages |

---

## Example Configuration

```yaml
messageQueue:
  url: amqp://user:password@rabbitmq:5672/
  exchange:
    name: domain.events
    type: topic
  publisher:
    confirms: true
    deliveryMode: 2
  consumer:
    prefetch: 10
    deadLetterQueue: dead-letter-queue
```

---

## Behavior / Constraints

### Delivery Semantics
- At-least-once delivery for consumers; implement idempotency

### Limits
- Max message size should be kept under 256KB for better performance

---

## Examples

### Example: Declaring Exchange (Node.js)

```javascript
await channel.assertExchange(process.env.EVENT_EXCHANGE || 'domain.events', process.env.EVENT_EXCHANGE_TYPE || 'topic', { durable: true });
```

---

## Related References

- [System Components](../architecture/system-components.md)

## See Also

- **How-To**: [Publish Events](../../how-to/integration/publish-events.md)
- **Explanation**: [Event-Driven Architecture](../../explanation/concepts/event-driven-architecture.md)

---

**Maintained By**: Architecture Team  
**Last Reviewed**: 2025-11-05  
**Version**: 1.0
