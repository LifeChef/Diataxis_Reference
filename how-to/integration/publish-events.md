# How to Publish Events

> **Document Type**: How-To Guide (Problem-Oriented)  
> **Category**: Integration  
> **Difficulty**: Moderate  
> **Last Updated**: 01/20/2025

## Problem Statement

Publish domain events from a service to the message queue (RabbitMQ) following team conventions.

---

## Prerequisites

Before you begin, ensure:

- [ ] Access to the message broker (RabbitMQ) and credentials
- [ ] Service can reach the broker host/port from its environment
- [ ] Environment variables for MQ are configured
- [ ] You understand the event schema for your domain

---

## Overview

This guide walks through initializing a publisher, declaring an exchange, and publishing an event payload with required headers.

**Time required**: 15–30 minutes

**Key steps**:
1. Install and import the client
2. Connect and declare exchange
3. Publish event payload with headers
4. Verify delivery

---

## Solution

### Step 1: Install client library

```bash
npm install amqplib
```

> Use the equivalent client in other languages if not using Node.js.

---

### Step 2: Connect and declare exchange

```javascript
const amqp = require('amqplib');

async function getChannel() {
  const conn = await amqp.connect(process.env.RABBITMQ_URL);
  const ch = await conn.createChannel();
  await ch.assertExchange(process.env.EVENT_EXCHANGE || 'domain.events', 'topic', { durable: true });
  return ch;
}
```

> 💡 Tip: Use durable exchanges and persistent messages for important events.

---

### Step 3: Publish an event

```javascript
async function publishUserRegistered(ch, payload) {
  const routingKey = 'user.registered.v1';
  const headers = {
    eventType: 'UserRegistered',
    version: '1.0',
    occurredAt: new Date().toISOString(),
  };

  const body = Buffer.from(JSON.stringify(payload));

  ch.publish(process.env.EVENT_EXCHANGE || 'domain.events', routingKey, body, {
    contentType: 'application/json',
    deliveryMode: 2, // persistent
    headers,
  });
}
```

> ⚠️ Warning: Never publish sensitive data (PII/secrets) in event payloads.

---

## Verification

1. Bind a temporary queue and check messages
   ```bash
   # Using rabbitmqadmin as an example
   rabbitmqadmin list exchanges name type
   rabbitmqadmin list bindings
   ```
2. Confirm consumers receive the event and process without error.

Expected output: event visible in broker management UI and consumed by at least one service.

---

## Troubleshooting

### Issue: Connection refused
- Cause: Wrong URL or network policy
- Solution: Verify `RABBITMQ_URL`, security groups, and service network access

### Issue: Messages not routed
- Cause: Missing binding or wrong routing key
- Solution: Confirm exchange type/topic and create appropriate bindings

---

## Alternative Approaches

1. Use a library wrapper/SDK shared by services
   - Pros: Consistent headers/retries
   - Cons: Shared dependency coupling

2. Use cloud-managed broker (e.g., Amazon MQ)
   - Pros: Operational simplicity
   - Cons: Vendor lock-in, cost

---

## Best Practices

- ✅ Use past-tense, specific event names (e.g., `UserRegistered`)
- ✅ Include version in routing key or headers
- ✅ Make messages idempotent on the consumer side
- ❌ Don’t block on publish (fire-and-forget)

---

## Related Documentation

- **Reference**: [Message Queue Configuration](../../reference/configuration/message-queue.md)
- **Explanation**: [Event-Driven Architecture](../../explanation/concepts/event-driven-architecture.md)
- **Tutorial**: [Building Microservices Tutorial](../../tutorials/advanced/building-microservices-tutorial.md)

---

**Author**: Documentation Team  
**Reviewed By**: Architecture Team  
**Last Updated**: 2025-11-05  
**Version**: 1.0
