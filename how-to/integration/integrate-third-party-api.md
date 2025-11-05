# How to Integrate a Third-Party API

> **Document Type**: How-To Guide (Problem-Oriented)  
> **Category**: Integration  
> **Difficulty**: Moderate

## Problem Statement

Connect your application to an external third-party API service.

---

## Prerequisites

- [ ] API key from third-party provider
- [ ] API documentation reviewed

---

## Solution

### Step 1: Install SDK

```bash
npm install third-party-sdk
```

---

### Step 2: Configure Credentials

Add to `.env`:

```
THIRD_PARTY_API_KEY=your_key_here
THIRD_PARTY_API_SECRET=your_secret_here
```

---

### Step 3: Create Client

```javascript
// src/services/thirdPartyClient.js
import ThirdPartySDK from 'third-party-sdk';

const client = new ThirdPartySDK({
  apiKey: process.env.THIRD_PARTY_API_KEY,
  apiSecret: process.env.THIRD_PARTY_API_SECRET
});

export default client;
```

---

### Step 4: Implement API Call

```javascript
import client from './services/thirdPartyClient';

async function fetchData() {
  try {
    const result = await client.getData();
    return result;
  } catch (error) {
    console.error('API call failed:', error);
    throw error;
  }
}
```

---

## Verification

Test the integration:

```bash
npm run test:integration
```

---

## Best Practices

- ✅ Handle rate limiting
- ✅ Implement retry logic
- ✅ Log API errors
- ❌ Don't expose API keys in code

---

**Related**: [API Client Reference](../../reference/api/rest-endpoints.md)
