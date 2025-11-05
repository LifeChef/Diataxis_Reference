# System Components Reference

> **Document Type**: Reference (Information-Oriented)  
> **Category**: Architecture  
> **Version**: 2.0  
> **Last Updated**: 01/20/2025

## Overview

Technical description of all system components and their relationships.

---

## Component Diagram

```
┌─────────────┐
│   Client    │
│ (Browser)   │
└──────┬──────┘
       │
       ↓
┌──────────────┐
│  API Gateway │ (Port 3000)
└──────┬───────┘
       │
       ├─────────────┬─────────────┬───────────────┐
       ↓             ↓             ↓               ↓
┌─────────┐   ┌──────────┐  ┌──────────┐  ┌─────────────┐
│  Auth   │   │   User   │  │  Order   │  │  Notification│
│ Service │   │ Service  │  │ Service  │  │   Service    │
└────┬────┘   └────┬─────┘  └────┬─────┘  └──────┬──────┘
     │             │              │                │
     └─────────────┴──────────────┴────────────────┘
                          │
                          ↓
                   ┌────────────┐
                   │  Database  │
                   │ PostgreSQL │
                   └────────────┘
```

---

## Frontend Components

### Web Application

**Technology**: React 18  
**Port**: 3001  
**Responsibilities**:
- User interface rendering
- Client-side routing
- State management
- API communication

**Key Files**:
- `src/App.jsx` - Main application
- `src/components/` - React components
- `src/services/` - API clients

---

## Backend Components

### API Gateway

**Technology**: Node.js + Express  
**Port**: 3000  
**Responsibilities**:
- Request routing
- Authentication
- Rate limiting
- Request/response transformation

**Endpoints**: See [REST API Reference](../api/rest-endpoints.md)

---

### Auth Service

**Technology**: Node.js  
**Port**: 3100  
**Responsibilities**:
- User authentication
- JWT token generation
- OAuth integration
- Password management

**Database Tables**:
- `users`
- `sessions`
- `oauth_tokens`

---

### User Service

**Technology**: Node.js  
**Port**: 3101  
**Responsibilities**:
- User profile management
- User preferences
- Profile data CRUD

**Database Tables**:
- `user_profiles`
- `user_preferences`

---

### Order Service

**Technology**: Node.js  
**Port**: 3102  
**Responsibilities**:
- Order processing
- Inventory management
- Payment integration

**Database Tables**:
- `orders`
- `order_items`
- `inventory`

---

### Notification Service

**Technology**: Node.js  
**Port**: 3103  
**Responsibilities**:
- Email notifications
- Push notifications
- SMS messaging
- Notification templates

**External Dependencies**:
- SendGrid (email)
- Firebase (push)
- Twilio (SMS)

---

## Data Layer

### PostgreSQL Database

**Version**: 14.x  
**Port**: 5432  
**Schemas**:
- `public` - Main application data
- `auth` - Authentication data
- `audit` - Audit logs

**Connection Pool**:
- Min: 10 connections
- Max: 100 connections
- Timeout: 30 seconds

---

### Redis Cache

**Version**: 7.x  
**Port**: 6379  
**Use Cases**:
- Session storage
- API response caching
- Rate limiting counters
- Temporary data

**Key Patterns**:
- `session:{userId}` - User sessions
- `cache:{endpoint}:{params}` - API cache
- `rate:{userId}:{endpoint}` - Rate limits

---

## Infrastructure Components

### Load Balancer

**Type**: Nginx  
**Responsibilities**:
- Traffic distribution
- SSL termination
- Health checks

---

### Message Queue

**Technology**: RabbitMQ  
**Port**: 5672  
**Queues**:
- `email-queue` - Email jobs
- `processing-queue` - Background jobs
- `dead-letter-queue` - Failed messages

---

## Monitoring

### Logging

**Tool**: Winston + ELK Stack  
**Levels**: error, warn, info, debug  
**Retention**: 30 days

---

### Metrics

**Tool**: Prometheus + Grafana  
**Metrics**:
- Request count/latency
- Error rates
- Database connections
- Memory/CPU usage

---

### Health Checks

Each service exposes `/health`:

**Response**:
```json
{
  "status": "healthy",
  "version": "2.0.1",
  "uptime": 86400,
  "dependencies": {
    "database": "healthy",
    "cache": "healthy"
  }
}
```

---

## Deployment

**Container Runtime**: Docker  
**Orchestration**: Kubernetes  
**Environments**:
- Development (local)
- Staging (staging cluster)
- Production (production cluster)

---

**See Also**:  
- [Explanation: Why Microservices](../../explanation/decisions/why-we-chose-microservices.md)
- [How-To: Deploy to Production](../../how-to/deployment/deploy-to-production.md)
