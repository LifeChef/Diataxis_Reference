# Tutorial: Building Microservices

> **Document Type**: Tutorial (Learning-Oriented)  
> **Audience**: Advanced  
> **Estimated Time**: 90 minutes

## Overview

Build a complete microservices architecture with API gateway and service communication.

**What you'll learn:**
- Microservices architecture
- Service-to-service communication
- API gateway pattern

---

## Prerequisites

- [ ] Docker installed
- [ ] Kubernetes basics
- [ ] Completed beginner tutorials

---

## Step 1: Create First Service

Create `user-service/`:

```javascript
// user-service/server.js
const express = require('express');
const app = express();

app.get('/users/:id', (req, res) => {
  res.json({ id: req.params.id, name: 'John Doe' });
});

app.listen(3001);
```

---

## Step 2: Create Second Service

Create `order-service/` following similar pattern.

---

## Step 3: Setup API Gateway

Configure routing and load balancing.

---

## Summary

You've built a microservices system!
