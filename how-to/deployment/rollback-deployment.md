# How to Rollback a Deployment

> **Document Type**: How-To Guide (Problem-Oriented)  
> **Category**: Deployment  
> **Difficulty**: Easy

## Problem Statement

Quickly rollback to a previous working version when production issues occur.

---

## Prerequisites

- [ ] Production access
- [ ] Previous deployment version known

---

## Solution

### Step 1: Identify Previous Version

```bash
kubectl rollout history deployment/app
```

Note the revision number to rollback to.

---

### Step 2: Execute Rollback

```bash
kubectl rollout undo deployment/app --to-revision=3
```

Replace `3` with your target revision.

---

### Step 3: Verify Rollback

```bash
kubectl rollout status deployment/app
```

Wait for "successfully rolled out" message.

---

## Verification

Check application is working:
```bash
curl https://api.yourapp.com/health
```

---

## Troubleshooting

### Issue: Rollback fails
**Solution**: Check pod status with `kubectl get pods`

---

**Related**: [Deploy to Production](deploy-to-production.md)
