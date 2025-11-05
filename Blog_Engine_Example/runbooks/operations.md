# Operations Runbook

> **Document Type**: Runbook (SAMPLE)  
> **Audience**: DevOps Engineers, On-Call Engineers, SRE  
> **Last Updated**: November 5, 2024

---

## Overview

This runbook provides operational procedures for the Blog Engine, including monitoring, incident response, troubleshooting, and maintenance tasks.

---

## Quick Reference

### Emergency Contacts
- **On-Call Engineer**: PagerDuty rotation
- **Tech Lead**: techlead@blogengine.com
- **DevOps Lead**: devops-lead@blogengine.com
- **CTO**: cto@blogengine.com

### Critical Systems
- **API Gateway**: Port 3000
- **Auth Service**: Port 3001
- **Post Service**: Port 3002
- **Database**: PostgreSQL (port 5432)
- **Cache**: Redis (port 6379)
- **Message Queue**: RabbitMQ (port 5672)

---

## System Health Checks

### Manual Health Check
```bash
# Check all service health endpoints
./scripts/health-check-all.sh

# Individual service checks
curl https://api.blogengine.com/health
curl https://api.blogengine.com/auth/health
curl https://api.blogengine.com/posts/health
```

### Expected Responses
```json
{
  "status": "healthy",
  "version": "1.2.0",
  "uptime": 86400,
  "dependencies": {
    "database": "healthy",
    "cache": "healthy",
    "messageQueue": "healthy"
  }
}
```

---

## Monitoring & Alerts

### Key Dashboards

**Grafana Dashboards**:
- **System Overview**: http://grafana.internal/d/system-overview
- **Service Metrics**: http://grafana.internal/d/services
- **Database Performance**: http://grafana.internal/d/database
- **Error Rates**: http://grafana.internal/d/errors

### Critical Alerts

| Alert | Threshold | Action |
|-------|-----------|--------|
| High Error Rate | > 5% | Investigate logs, check for deployment |
| Slow Response Time | P95 > 2s | Check database, review slow queries |
| Service Down | Any service unhealthy > 2 min | Restart service, check logs |
| Database Connection Pool | > 90% used | Scale connections, check for leaks |
| Disk Space | > 85% used | Clean logs, scale storage |
| Memory Usage | > 90% | Investigate memory leaks, restart |

### Alert Channels
- **PagerDuty**: Critical alerts (P0/P1)
- **Slack #alerts**: Warning alerts (P2/P3)
- **Email**: Daily summaries

---

## Common Incidents

### 1. Service is Down

**Symptoms**: Health check failing, 503 errors

**Diagnosis**:
```bash
# Check service status
kubectl get pods -n production

# View recent logs
kubectl logs -n production deployment/blog-engine --tail=100

# Check resource usage
kubectl top pods -n production
```

**Resolution**:
```bash
# Restart the service
kubectl rollout restart deployment/blog-engine -n production

# Monitor restart
kubectl rollout status deployment/blog-engine -n production

# Verify health
curl https://api.blogengine.com/health
```

---

### 2. High Response Times

**Symptoms**: API response time > 2 seconds

**Diagnosis**:
```bash
# Check database slow queries
psql -U admin -d blogengine -c "
  SELECT query, mean_exec_time, calls
  FROM pg_stat_statements
  ORDER BY mean_exec_time DESC
  LIMIT 10;"

# Check cache hit rate
redis-cli INFO stats | grep keyspace_hits

# Check connection pools
kubectl exec -it deployment/blog-engine -- npm run pool:status
```

**Resolution**:
- Add database indexes for slow queries
- Increase cache TTL for frequently accessed data
- Scale service horizontally if CPU/memory high
- Review and optimize N+1 queries

---

### 3. Database Connection Errors

**Symptoms**: "connection pool exhausted" errors

**Diagnosis**:
```bash
# Check active connections
psql -U admin -d blogengine -c "
  SELECT count(*) FROM pg_stat_activity;"

# Check connection pool config
kubectl exec -it deployment/blog-engine -- cat config/database.json
```

**Resolution**:
```bash
# Increase connection pool size (temporary)
kubectl set env deployment/blog-engine DB_POOL_SIZE=50

# Kill long-running queries
psql -U admin -d blogengine -c "
  SELECT pg_terminate_backend(pid)
  FROM pg_stat_activity
  WHERE state = 'idle' AND state_change < NOW() - INTERVAL '10 minutes';"

# Add read replicas for read-heavy workloads
```

---

### 4. High Memory Usage

**Symptoms**: Pods restarting, OOMKilled events

**Diagnosis**:
```bash
# Check memory usage
kubectl top pods -n production

# Review memory limits
kubectl describe pod <pod-name> -n production

# Heap dump (Node.js)
kubectl exec -it <pod-name> -- npm run heapdump
```

**Resolution**:
```bash
# Increase memory limits
kubectl set resources deployment/blog-engine \
  --limits=memory=2Gi \
  --requests=memory=1Gi

# Review for memory leaks in code
# Analyze heap dumps with Chrome DevTools
```

---

## Maintenance Tasks

### Daily

**Automated**:
- Database backups (2 AM UTC)
- Log rotation
- Metrics collection

**Manual** (if issues):
- Review error logs for new patterns
- Check disk space on database servers

---

### Weekly

- [ ] Review slow query log
- [ ] Check for security updates
- [ ] Verify backup integrity
- [ ] Review capacity metrics

```bash
# Weekly maintenance script
./scripts/weekly-maintenance.sh
```

---

### Monthly

- [ ] Database vacuum and analyze
- [ ] Review and archive old logs
- [ ] SSL certificate expiration check
- [ ] Dependency security audit

```bash
# Monthly maintenance
./scripts/monthly-maintenance.sh

# Database maintenance
psql -U admin -d blogengine -c "VACUUM ANALYZE;"
```

---

### Quarterly

- [ ] Load testing
- [ ] Disaster recovery drill
- [ ] Infrastructure cost review
- [ ] Security audit
- [ ] Rotate secrets and API keys

---

## Backup & Recovery

### Automated Backups

**Database**:
- **Frequency**: Daily at 2 AM UTC
- **Retention**: 30 days
- **Location**: AWS S3 (`s3://blogengine-backups/db/`)

**Configuration**:
- **Frequency**: Weekly
- **Retention**: 90 days
- **Location**: Git repository (encrypted)

### Restore Database

```bash
# List available backups
aws s3 ls s3://blogengine-backups/db/

# Download backup
aws s3 cp s3://blogengine-backups/db/backup-November 5, 2024.sql.gz .

# Restore (CAUTION: This will overwrite current data)
gunzip backup-November 5, 2024.sql.gz
psql -U admin -d blogengine < backup-November 5, 2024.sql

# Verify restore
psql -U admin -d blogengine -c "SELECT COUNT(*) FROM posts;"
```

### Point-in-Time Recovery

```bash
# Restore to specific timestamp
pg_basebackup -D /var/lib/postgresql/data \
  --target-time='November 5, 2024 14:30:00'
```

---

## Scaling Operations

### Horizontal Scaling

**Scale services**:
```bash
# Scale Post Service to 5 replicas
kubectl scale deployment/post-service --replicas=5 -n production

# Auto-scaling configuration
kubectl autoscale deployment/post-service \
  --min=3 --max=10 \
  --cpu-percent=70 -n production
```

### Database Scaling

**Add read replica**:
```bash
# Provision read replica (AWS RDS)
aws rds create-db-instance-read-replica \
  --db-instance-identifier blogengine-read-replica \
  --source-db-instance-identifier blogengine-primary

# Update application config to use read replica
kubectl set env deployment/blog-engine \
  DB_READ_HOST=blogengine-read-replica.aws.com
```

---

## Log Management

### Access Logs

```bash
# View live logs
kubectl logs -f deployment/blog-engine -n production

# Search logs with grep
kubectl logs deployment/blog-engine -n production | grep ERROR

# Access Elasticsearch logs (ELK stack)
# https://kibana.internal/app/discover
```

### Log Levels
- **ERROR**: Critical issues requiring immediate attention
- **WARN**: Potential problems to investigate
- **INFO**: Normal operational messages
- **DEBUG**: Detailed diagnostic information

---

## Security Operations

### Rotate Secrets

```bash
# Rotate database password
./scripts/rotate-db-password.sh

# Update JWT secret
./scripts/rotate-jwt-secret.sh

# Rotate API keys
./scripts/rotate-api-keys.sh
```

### Security Scan

```bash
# Run vulnerability scan
npm audit

# Container security scan
docker scan blogengine/api:latest

# OWASP ZAP scan
./scripts/security-scan.sh
```

---

## Disaster Recovery

### Recovery Time Objective (RTO)
- **Critical Services**: 15 minutes
- **Database**: 1 hour
- **Full System**: 4 hours

### Recovery Point Objective (RPO)
- **Database**: 1 hour (continuous WAL archiving)
- **File Storage**: 24 hours

### DR Procedure

1. **Assess Impact**: Determine scope of disaster
2. **Activate DR Plan**: Notify team, declare incident
3. **Restore Services**: Follow service-specific runbooks
4. **Restore Data**: Use latest backup or point-in-time recovery
5. **Verify System**: Run smoke tests
6. **Post-Mortem**: Document incident and improvements

---

## Performance Tuning

### Database Optimization

```sql
-- Identify missing indexes
SELECT schemaname, tablename, attname, n_distinct, correlation
FROM pg_stats
WHERE schemaname = 'public'
  AND n_distinct > 100
  AND correlation < 0.1;

-- Create index for slow query
CREATE INDEX idx_posts_author_created 
ON posts(author_id, created_at DESC);

-- Analyze table statistics
ANALYZE posts;
```

### Cache Optimization

```bash
# Check Redis memory usage
redis-cli INFO memory

# Review cache hit rate
redis-cli INFO stats | grep keyspace

# Increase cache TTL for static content
# Update in application config
```

---

## Related Documentation

- [Deployment Guide](deployment.md)
- [Incident Response](incident-response.md)
- [Monitoring Guide](monitoring.md)
- [System Architecture](../docs/architecture/system-overview.md)

---

**Maintained By**: DevOps Team  
**On-Call Rotation**: PagerDuty  
**Review Frequency**: Monthly  
**Next Review**: December 5, 2024
