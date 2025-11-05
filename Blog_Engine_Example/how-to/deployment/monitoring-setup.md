# Configure Monitoring

> **Document Type**: How-To Guide (SAMPLE)  
> **Audience**: DevOps Engineers, SRE  
> **Difficulty**: Advanced  
> **Last Updated**: November 5, 2024

---

## Problem

You need to set up comprehensive monitoring for the Blog Engine using Prometheus, Grafana, and alerting.

---

## Prerequisites

- Kubernetes cluster running
- Helm 3 installed
- kubectl configured
- Basic understanding of Prometheus/Grafana

---

## Solution

### Step 1: Install Prometheus using Helm

```bash
# Add Prometheus Helm repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Create namespace
kubectl create namespace monitoring

# Install Prometheus stack (includes Grafana)
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set prometheus.prometheusSpec.retention=30d \
  --set grafana.adminPassword=admin123
```

---

### Step 2: Expose Metrics in Your Application

Add Prometheus client to your Node.js service:

```bash
npm install prom-client
```

Create `src/monitoring/metrics.js`:

```javascript
const promClient = require('prom-client');

// Create a Registry
const register = new promClient.Registry();

// Add default metrics
promClient.collectDefaultMetrics({ register });

// Custom metrics
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.5, 1, 2, 5]
});

const httpRequestTotal = new promClient.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

const activeConnections = new promClient.Gauge({
  name: 'active_connections',
  help: 'Number of active connections'
});

const databaseQueryDuration = new promClient.Histogram({
  name: 'database_query_duration_seconds',
  help: 'Duration of database queries',
  labelNames: ['operation', 'table'],
  buckets: [0.01, 0.05, 0.1, 0.5, 1, 2]
});

// Register custom metrics
register.registerMetric(httpRequestDuration);
register.registerMetric(httpRequestTotal);
register.registerMetric(activeConnections);
register.registerMetric(databaseQueryDuration);

module.exports = {
  register,
  httpRequestDuration,
  httpRequestTotal,
  activeConnections,
  databaseQueryDuration
};
```

---

### Step 3: Add Metrics Middleware

Create `src/middleware/metrics.middleware.js`:

```javascript
const { httpRequestDuration, httpRequestTotal } = require('../monitoring/metrics');

module.exports = (req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    const route = req.route?.path || req.path;
    
    httpRequestDuration.observe(
      {
        method: req.method,
        route: route,
        status_code: res.statusCode
      },
      duration
    );

    httpRequestTotal.inc({
      method: req.method,
      route: route,
      status_code: res.statusCode
    });
  });

  next();
};
```

Add to Express app:

```javascript
const express = require('express');
const metricsMiddleware = require('./middleware/metrics.middleware');
const { register } = require('./monitoring/metrics');

const app = express();

// Add metrics middleware
app.use(metricsMiddleware);

// Metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});
```

---

### Step 4: Configure ServiceMonitor

Create `k8s/production/servicemonitor.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: blog-engine-metrics
  namespace: production
  labels:
    app: blog-engine
spec:
  ports:
  - name: metrics
    port: 3000
    targetPort: 3000
  selector:
    app: blog-engine
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: blog-engine
  namespace: production
  labels:
    app: blog-engine
spec:
  selector:
    matchLabels:
      app: blog-engine
  endpoints:
  - port: metrics
    path: /metrics
    interval: 30s
```

Apply:
```bash
kubectl apply -f k8s/production/servicemonitor.yaml
```

---

### Step 5: Import Grafana Dashboards

Access Grafana:

```bash
# Port-forward Grafana service
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
```

Navigate to `http://localhost:3000` (admin/admin123)

Import dashboards:
1. Click "+" → "Import"
2. Enter dashboard ID or upload JSON
3. Recommended dashboards:
   - **Node Exporter**: 1860
   - **Kubernetes Cluster**: 7249
   - **PostgreSQL**: 9628

---

### Step 6: Create Custom Dashboard

Create `monitoring/grafana-dashboard.json`:

```json
{
  "dashboard": {
    "title": "Blog Engine - Application Metrics",
    "panels": [
      {
        "title": "HTTP Request Rate",
        "targets": [{
          "expr": "rate(http_requests_total[5m])"
        }],
        "type": "graph"
      },
      {
        "title": "Request Duration (P95)",
        "targets": [{
          "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))"
        }],
        "type": "graph"
      },
      {
        "title": "Error Rate",
        "targets": [{
          "expr": "rate(http_requests_total{status_code=~\"5..\"}[5m])"
        }],
        "type": "graph"
      },
      {
        "title": "Database Query Duration",
        "targets": [{
          "expr": "rate(database_query_duration_seconds_sum[5m]) / rate(database_query_duration_seconds_count[5m])"
        }],
        "type": "graph"
      }
    ]
  }
}
```

Import via Grafana UI.

---

### Step 7: Configure Alerting

Create `k8s/production/alertrules.yaml`:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: blog-engine-alerts
  namespace: production
spec:
  groups:
  - name: blog-engine
    interval: 30s
    rules:
    - alert: HighErrorRate
      expr: |
        rate(http_requests_total{status_code=~"5.."}[5m]) > 0.05
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "High error rate detected"
        description: "Error rate is {{ $value }} requests/sec"

    - alert: HighResponseTime
      expr: |
        histogram_quantile(0.95, 
          rate(http_request_duration_seconds_bucket[5m])
        ) > 2
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High response time detected"
        description: "P95 response time is {{ $value }}s"

    - alert: ServiceDown
      expr: up{job="blog-engine"} == 0
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: "Service is down"
        description: "{{ $labels.instance }} is down"

    - alert: HighMemoryUsage
      expr: |
        (container_memory_usage_bytes{pod=~"blog-engine.*"} 
         / container_spec_memory_limit_bytes{pod=~"blog-engine.*"}) > 0.9
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High memory usage"
        description: "Memory usage is {{ $value | humanizePercentage }}"
```

Apply:
```bash
kubectl apply -f k8s/production/alertrules.yaml
```

---

### Step 8: Configure Alert Notifications

Configure Alertmanager for Slack notifications:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: alertmanager-config
  namespace: monitoring
data:
  alertmanager.yml: |
    global:
      slack_api_url: 'YOUR_SLACK_WEBHOOK_URL'
    
    route:
      group_by: ['alertname', 'severity']
      group_wait: 10s
      group_interval: 5m
      repeat_interval: 12h
      receiver: 'slack-notifications'
      routes:
      - match:
          severity: critical
        receiver: 'pagerduty'

    receivers:
    - name: 'slack-notifications'
      slack_configs:
      - channel: '#alerts'
        title: '{{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
    
    - name: 'pagerduty'
      pagerduty_configs:
      - service_key: 'YOUR_PAGERDUTY_KEY'
```

---

## Verification

### Check Metrics Collection

```bash
# Port-forward Prometheus
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090

# Navigate to http://localhost:9090
# Query: http_requests_total
```

### Check Grafana Dashboards

```bash
# Access Grafana
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80

# Navigate to http://localhost:3000
# View dashboards
```

### Test Alerts

Trigger an alert:

```bash
# Generate high load
ab -n 10000 -c 100 https://api.blogengine.com/posts

# Check Alertmanager
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-alertmanager 9093:9093
# Navigate to http://localhost:9093
```

---

## Troubleshooting

**Metrics not appearing:**
- Check `/metrics` endpoint is accessible
- Verify ServiceMonitor selector matches service labels
- Review Prometheus targets: `Status → Targets`

**Alerts not firing:**
- Verify PrometheusRule applied: `kubectl get prometheusrules -n production`
- Check alert expression in Prometheus UI
- Review Alertmanager logs

**Grafana can't connect to Prometheus:**
- Check Prometheus service name
- Verify namespace is correct
- Test connection from Grafana pod

---

## Related

- [Operations Runbook](../../runbooks/operations.md)
- [CI/CD Setup](cicd-setup.md)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
