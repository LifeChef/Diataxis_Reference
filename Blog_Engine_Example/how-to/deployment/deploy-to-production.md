# How to Deploy to Production

> **Document Type**: How-To Guide (Problem-Oriented)  
> **Category**: Deployment  
> **Difficulty**: Moderate  
> **Last Updated**: November 5, 2024

## Problem Statement

Deploy the Blog Engine application to production Kubernetes cluster with zero downtime.

---

## Prerequisites

Before you begin, ensure:

- [ ] Access to production Kubernetes cluster
- [ ] `kubectl` installed and configured
- [ ] Docker images built and pushed to registry
- [ ] Database migrations prepared
- [ ] Production `.env` files configured
- [ ] Rollback plan documented

---

## Overview

This guide walks through deploying to production using our CI/CD pipeline and Kubernetes.

**Time required**: 30-45 minutes

**Key steps**:
1. Pre-deployment checks
2. Run database migrations
3. Deploy services to Kubernetes
4. Verify deployment health
5. Monitor for issues

---

## Solution

### Step 1: Pre-Deployment Checks

Run automated checks before deploying.

```bash
# Verify all tests pass
npm run test:all

# Check Docker images exist
./scripts/check-images.sh v1.2.0

# Verify migrations are ready
npm run migrate:dry-run

# Check production config
./scripts/validate-config.sh production
```

**Expected output**: All checks pass with no errors.

> ⚠️ **Warning**: Do not proceed if any checks fail. Investigate and fix issues first.

---

### Step 2: Run Database Migrations

Apply database migrations before deploying new code.

```bash
# Set context to production
export NODE_ENV=production

# Run migrations (with backup)
npm run migrate:prod

# Verify schema version
npm run db:version
```

**Expected output**:
```
✓ Backup created: backup_20251105_100000.sql
✓ Running 3 new migrations
✓ Migration complete. Schema version: 1.2.0
```

> 💡 **Tip**: Migrations run automatically in a transaction. If any fail, all roll back.

---

### Step 3: Deploy to Kubernetes

Deploy services using kubectl or CI/CD pipeline.

**Option A: Using CI/CD (Recommended)**

1. Create a release tag:
   ```bash
   git tag -a v1.2.0 -m "Release v1.2.0"
   git push origin v1.2.0
   ```

2. GitHub Actions automatically:
   - Builds Docker images
   - Pushes to registry
   - Deploys to staging
   - Waits for approval
   - Deploys to production

3. Approve deployment in GitHub Actions UI

**Option B: Manual Deployment**

```bash
# Set Kubernetes context
kubectl config use-context production

# Apply configuration
kubectl apply -f k8s/production/

# Deploy services
kubectl set image deployment/api-gateway \
  api-gateway=blogengine/api-gateway:v1.2.0

kubectl set image deployment/auth-service \
  auth-service=blogengine/auth-service:v1.2.0

kubectl set image deployment/post-service \
  post-service=blogengine/post-service:v1.2.0

kubectl set image deployment/comment-service \
  comment-service=blogengine/comment-service:v1.2.0

kubectl set image deployment/notification-service \
  notification-service=blogengine/notification-service:v1.2.0
```

---

### Step 4: Verify Deployment

Check that all services are healthy.

```bash
# Watch rollout status
kubectl rollout status deployment/api-gateway
kubectl rollout status deployment/auth-service
kubectl rollout status deployment/post-service

# Check pod health
kubectl get pods -l app=blog-engine

# Test health endpoints
curl https://api.blogengine.com/health

# Check service connectivity
./scripts/smoke-test.sh production
```

**Expected output**:
```
deployment "api-gateway" successfully rolled out
deployment "auth-service" successfully rolled out
deployment "post-service" successfully rolled out

NAME                         READY   STATUS    AGE
api-gateway-7d4b9c8f-abc12   2/2     Running   1m
auth-service-6c9d8a-xyz34    2/2     Running   1m
post-service-5b8c7f-def56    2/2     Running   1m

Health check: ✓ All services healthy
Smoke tests: ✓ All passed
```

---

### Step 5: Monitor Post-Deployment

Monitor for 15-30 minutes after deployment.

```bash
# Watch metrics dashboard
open https://grafana.blogengine.com/d/production

# Monitor error rates
kubectl logs -f deployment/api-gateway --tail=100

# Check application logs
./scripts/tail-logs.sh production api-gateway
```

**Key metrics to watch**:
- Error rate < 1%
- Response time < 200ms p99
- CPU usage < 70%
- Memory usage < 80%
- No database connection errors

---

## Verification Checklist

After deployment, verify:

- [ ] All pods are running and healthy
- [ ] Health endpoints return 200 OK
- [ ] Database migrations completed
- [ ] Smoke tests pass
- [ ] Error rates normal
- [ ] Response times acceptable
- [ ] No alerts triggered
- [ ] Metrics dashboard shows healthy state

---

## Troubleshooting

### Issue: Pods stuck in "ImagePullBackOff"

**Cause**: Docker image not found in registry

**Solution**:
```bash
# Check image exists
docker pull blogengine/api-gateway:v1.2.0

# If missing, rebuild and push
./scripts/build-and-push.sh v1.2.0

# Restart deployment
kubectl rollout restart deployment/api-gateway
```

---

### Issue: Migration fails

**Cause**: Schema conflict or data integrity issue

**Solution**:
```bash
# Check migration logs
npm run migrate:status

# Rollback migration
npm run migrate:rollback

# Fix migration file
# Re-run: npm run migrate:prod
```

---

### Issue: High error rate after deployment

**Cause**: New code issue or configuration problem

**Solution**:
```bash
# Immediate rollback
kubectl rollout undo deployment/api-gateway

# Check logs for errors
kubectl logs deployment/api-gateway --tail=500 | grep ERROR

# If critical, full rollback
./scripts/rollback.sh v1.1.0
```

---

## Rollback Procedure

If deployment fails:

```bash
# Option 1: Rollback last deployment
kubectl rollout undo deployment/api-gateway

# Option 2: Rollback to specific version
kubectl rollout undo deployment/api-gateway --to-revision=5

# Option 3: Full system rollback
./scripts/rollback.sh v1.1.0

# Verify rollback
kubectl rollout status deployment/api-gateway
```

---

## Best Practices

- ✅ **Do**: Deploy during low-traffic periods (2-4 AM)
- ✅ **Do**: Have team members available during deployment
- ✅ **Do**: Test rollback procedure in staging first
- ✅ **Do**: Monitor metrics for 30 minutes post-deployment
- ✅ **Do**: Document any issues in postmortem

- ❌ **Don't**: Deploy on Fridays unless critical
- ❌ **Don't**: Skip pre-deployment checks
- ❌ **Don't**: Deploy multiple services simultaneously
- ❌ **Don't**: Leave deployment unmonitored

---

## Post-Deployment Tasks

After successful deployment:

1. **Update documentation** if needed
2. **Notify stakeholders** of deployment completion
3. **Update release notes** in GitHub
4. **Archive deployment logs**
5. **Schedule postmortem** if issues occurred

---

## Related Documentation

- **Reference**: [System Architecture](../../reference/architecture/system-architecture.md)
- **How-To**: [Rollback Deployment](rollback-deployment.md)
- **How-To**: [Configure Monitoring](monitoring-setup.md)
- **Explanation**: [CI/CD Pipeline](../../explanation/concepts/cicd-pipeline.md)

---

**Author**: DevOps Team  
**Reviewed By**: Tech Lead  
**Last Updated**: November 5, 2024  
**Version**: 1.0
