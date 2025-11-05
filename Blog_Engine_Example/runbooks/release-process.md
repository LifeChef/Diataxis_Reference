# Release Process

> **Document Type**: Runbook (SAMPLE)  
> **Audience**: DevOps, Engineering Managers, Release Managers  
> **Last Updated**: November 5, 2024

---

## Overview

This document outlines the standard release process for the Blog Engine, including versioning, release cadence, deployment procedures, and rollback strategies.

---

## Release Cadence

### Regular Releases
- **Weekly Releases**: Minor features and bug fixes (Fridays, 2 PM UTC)
- **Patch Releases**: Critical bug fixes (as needed, any day)
- **Major Releases**: Breaking changes, new major features (quarterly)

### Release Schedule
```
Week 1-2: Development
Week 3: Code freeze, testing
Week 4: Release preparation
Friday 2 PM: Deploy to production
```

---

## Versioning Strategy

We follow **Semantic Versioning** (SemVer): `MAJOR.MINOR.PATCH`

### Version Increments
- **MAJOR** (1.0.0 → 2.0.0): Breaking API changes, major architecture changes
- **MINOR** (1.1.0 → 1.2.0): New features, backward compatible
- **PATCH** (1.1.1 → 1.1.2): Bug fixes, security patches

### Examples
```
1.0.0 - Initial release
1.1.0 - Added notification system
1.1.1 - Fixed email delivery bug
2.0.0 - New API v2, deprecated v1 endpoints
```

---

## Pre-Release Checklist

### 1 Week Before Release
- [ ] All feature PRs merged to `main`
- [ ] Release notes drafted
- [ ] Breaking changes documented
- [ ] Database migrations reviewed and tested
- [ ] Changelog updated

### 3 Days Before Release
- [ ] Code freeze initiated
- [ ] QA testing completed
- [ ] Performance testing completed
- [ ] Security scan passed
- [ ] All tests passing (unit, integration, E2E)

### 1 Day Before Release
- [ ] Staging deployment successful
- [ ] Smoke tests on staging passed
- [ ] Rollback plan documented
- [ ] On-call team notified
- [ ] Stakeholders notified

---

## Release Steps

### Step 1: Version Bump
```bash
# Update version in package.json
npm version minor  # or major/patch

# Tag the release
git tag -a v1.2.0 -m "Release version 1.2.0"
git push origin v1.2.0
```

### Step 2: Build and Test
```bash
# Run full test suite
npm run test:all

# Build production artifacts
npm run build:production

# Verify build
npm run verify:build
```

### Step 3: Deploy to Staging
```bash
# Deploy to staging environment
kubectl apply -f k8s/staging/

# Wait for rollout
kubectl rollout status deployment/blog-engine -n staging

# Run smoke tests
npm run test:smoke -- --env=staging
```

### Step 4: Production Deployment
```bash
# Create deployment record
./scripts/record-deployment.sh v1.2.0

# Deploy to production (blue-green deployment)
kubectl apply -f k8s/production/

# Monitor rollout
kubectl rollout status deployment/blog-engine -n production

# Verify health checks
./scripts/health-check.sh production
```

### Step 5: Post-Deployment
```bash
# Run smoke tests on production
npm run test:smoke -- --env=production

# Monitor error rates (15 minutes)
./scripts/monitor-errors.sh

# Notify stakeholders
./scripts/notify-release.sh v1.2.0
```

---

## Rollback Procedure

### Automatic Rollback Triggers
- Error rate > 5%
- Response time > 2 seconds (P95)
- Failed health checks
- Critical service down

### Manual Rollback Steps
```bash
# Immediate rollback to previous version
kubectl rollout undo deployment/blog-engine -n production

# Verify rollback
kubectl rollout status deployment/blog-engine -n production

# Notify team
./scripts/notify-rollback.sh v1.2.0

# Create incident report
./scripts/create-incident.sh
```

### Rollback Decision Time
- **0-5 minutes**: Monitor for issues
- **5-10 minutes**: If issues persist, initiate rollback
- **10+ minutes**: Rollback complete, investigate

---

## Database Migrations

### Forward Migration (Deployment)
```bash
# Backup database before migration
./scripts/backup-db.sh production

# Run migrations
npm run migrate:up -- --env=production

# Verify migration
npm run migrate:status -- --env=production
```

### Rollback Migration
```bash
# Rollback migrations if deployment fails
npm run migrate:down -- --env=production --steps=1

# Restore from backup if needed
./scripts/restore-db.sh production latest
```

---

## Communication Plan

### Before Release
- **T-24 hours**: Email to all stakeholders
- **T-4 hours**: Slack notification in #releases channel
- **T-1 hour**: Final reminder, on-call team ready

### During Release
- **Start**: "Deployment started" message
- **Every 5 min**: Progress updates
- **Complete**: "Deployment successful" with verification results

### After Release
- **T+15 min**: Monitoring report
- **T+1 hour**: Full release summary
- **Next day**: Release retrospective scheduled

---

## Release Metrics

### Key Metrics to Track
- **Deployment Duration**: Target < 20 minutes
- **Rollback Rate**: Target < 5%
- **Deployment Frequency**: Target 1/week minimum
- **Mean Time to Recovery**: Target < 15 minutes
- **Change Failure Rate**: Target < 10%

### Post-Release Monitoring (24 hours)
- Error rates
- Response times
- Database performance
- User-reported issues
- Resource utilization

---

## Emergency Hotfix Process

### When to Use
- Critical security vulnerability
- Data loss or corruption risk
- Complete service outage
- Major user-impacting bug

### Hotfix Steps
1. Create hotfix branch from production tag
2. Fix issue with minimal changes
3. Fast-track review (1 reviewer minimum)
4. Deploy directly to production
5. Backport fix to main branch
6. Document in changelog

```bash
# Create hotfix branch
git checkout -b hotfix/v1.2.1 v1.2.0

# Make fix and commit
git commit -m "fix: critical authentication bug"

# Tag and deploy
git tag v1.2.1
./scripts/deploy.sh v1.2.1 production

# Merge back to main
git checkout main
git merge hotfix/v1.2.1
```

---

## Approval Matrix

| Release Type | Approvals Required | Timeline |
|--------------|-------------------|----------|
| Patch Release | 1 Tech Lead | Same day |
| Minor Release | 1 Tech Lead + 1 EM | 1 week |
| Major Release | CTO + Product | 1 month |
| Emergency Hotfix | On-call Lead | Immediate |

---

## Related Documentation

- [Deployment Guide](deployment.md)
- [Monitoring Guide](monitoring.md)
- [Incident Response](incident-response.md)
- [CONTRIBUTING.md](../CONTRIBUTING.md)

---

**Maintained By**: DevOps Team  
**Review Frequency**: Quarterly  
**Next Review**: February 5, 2025
