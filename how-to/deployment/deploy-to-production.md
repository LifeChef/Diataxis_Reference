# How to Deploy to Production

> **Document Type**: How-To Guide (Problem-Oriented)  
> **Category**: Deployment  
> **Difficulty**: Moderate

## Problem Statement

Deploy your application to the production environment safely and reliably.

---

## Prerequisites

- [ ] Production access credentials
- [ ] Application tested in staging
- [ ] Deployment checklist completed

---

## Solution

### Step 1: Build Production Assets

```bash
npm run build
```

This creates optimized production files in `dist/`.

---

### Step 2: Run Pre-deployment Checks

```bash
npm run test:prod
npm run lint
```

Ensure all tests pass before deploying.

---

### Step 3: Deploy to Production

```bash
./scripts/deploy.sh production
```

**Parameters:**
- `production`: Target environment

**Expected outcome**: Deployment completes with success message.

---

### Step 4: Verify Deployment

1. Check health endpoint:
   ```bash
   curl https://api.yourapp.com/health
   ```

2. Monitor logs:
   ```bash
   kubectl logs -f deployment/app
   ```

---

## Verification

- [ ] Application is accessible
- [ ] Health checks passing
- [ ] No errors in logs
- [ ] Database migrations completed

---

## Troubleshooting

### Issue: Deployment times out
**Solution**: Check network connectivity and increase timeout in deploy script.

### Issue: Health check fails
**Solution**: Verify environment variables and database connections.

---

## Best Practices

- ✅ Always deploy during low-traffic periods
- ✅ Have rollback plan ready
- ❌ Don't skip testing phase

---

**Related**: [Rollback Deployment](rollback-deployment.md)
