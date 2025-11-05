# Set Up CI/CD Pipeline

> **Document Type**: How-To Guide (SAMPLE)  
> **Audience**: DevOps Engineers, Tech Leads  
> **Difficulty**: Advanced  
> **Last Updated**: November 5, 2024

---

## Problem

You need to set up a continuous integration and continuous deployment (CI/CD) pipeline for the Blog Engine using GitHub Actions.

---

## Prerequisites

- GitHub repository access
- Docker Hub account
- Kubernetes cluster access
- AWS or cloud provider credentials

---

## Solution

### Step 1: Create GitHub Actions Workflow

Create `.github/workflows/ci-cd.yml`:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: docker.io
  IMAGE_NAME: blogengine/api

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: blog_test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
      
      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run type check
        run: npm run type-check

      - name: Run unit tests
        run: npm test
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/blog_test
          REDIS_URL: redis://localhost:6379

      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/blog_test

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          fail_ci_if_error: true

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' || github.ref == 'refs/heads/develop'
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=sha,prefix={{branch}}-
            type=semver,pattern={{version}}

      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'
    environment: staging
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Update kubeconfig
        run: |
          aws eks update-kubeconfig --name blog-engine-staging --region us-east-1

      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/blog-engine \
            blog-engine=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:develop-${{ github.sha }} \
            -n staging
          kubectl rollout status deployment/blog-engine -n staging --timeout=5m

      - name: Run smoke tests
        run: |
          npm run test:smoke -- --env=staging

  deploy-production:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Update kubeconfig
        run: |
          aws eks update-kubeconfig --name blog-engine-production --region us-east-1

      - name: Deploy to Kubernetes (Blue-Green)
        run: |
          # Tag new version as green
          kubectl set image deployment/blog-engine-green \
            blog-engine=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:main-${{ github.sha }} \
            -n production
          
          # Wait for green deployment
          kubectl rollout status deployment/blog-engine-green -n production --timeout=5m
          
          # Run health checks
          ./scripts/health-check.sh green
          
          # Switch traffic to green
          kubectl patch service blog-engine -n production \
            -p '{"spec":{"selector":{"version":"green"}}}'
          
          # Wait and verify
          sleep 60
          ./scripts/health-check.sh production
          
          # Update blue to new version (for next deploy)
          kubectl set image deployment/blog-engine-blue \
            blog-engine=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:main-${{ github.sha }} \
            -n production

      - name: Run smoke tests
        run: |
          npm run test:smoke -- --env=production

      - name: Notify deployment
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Production deployment completed!'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
        if: always()
```

---

### Step 2: Create Dockerfile

Create `Dockerfile` in project root:

```dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine

WORKDIR /app

# Copy only necessary files
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

USER nodejs

EXPOSE 3000

CMD ["node", "dist/index.js"]
```

---

### Step 3: Configure GitHub Secrets

Add these secrets to your GitHub repository:

```
Settings → Secrets and variables → Actions → New repository secret
```

Required secrets:
- `DOCKER_USERNAME`: Docker Hub username
- `DOCKER_PASSWORD`: Docker Hub token
- `AWS_ACCESS_KEY_ID`: AWS access key
- `AWS_SECRET_ACCESS_KEY`: AWS secret key
- `SLACK_WEBHOOK`: Slack webhook URL

---

### Step 4: Create Kubernetes Manifests

Create `k8s/production/deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blog-engine-blue
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: blog-engine
      version: blue
  template:
    metadata:
      labels:
        app: blog-engine
        version: blue
    spec:
      containers:
      - name: blog-engine
        image: blogengine/api:latest
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: blog-engine-secrets
              key: database-url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: blog-engine-secrets
              key: redis-url
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blog-engine-green
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: blog-engine
      version: green
  template:
    metadata:
      labels:
        app: blog-engine
        version: green
    spec:
      containers:
      - name: blog-engine
        image: blogengine/api:latest
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: blog-engine-secrets
              key: database-url
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
```

---

### Step 5: Create Health Check Script

Create `scripts/health-check.sh`:

```bash
#!/bin/bash

ENV=$1
TIMEOUT=300
INTERVAL=5

if [ "$ENV" = "production" ]; then
  URL="https://api.blogengine.com/health"
elif [ "$ENV" = "staging" ]; then
  URL="https://staging-api.blogengine.com/health"
elif [ "$ENV" = "green" ]; then
  URL="http://blog-engine-green.production.svc.cluster.local:3000/health"
else
  echo "Invalid environment: $ENV"
  exit 1
fi

echo "Health checking $URL..."

elapsed=0
while [ $elapsed -lt $TIMEOUT ]; do
  status_code=$(curl -s -o /dev/null -w "%{http_code}" $URL)
  
  if [ "$status_code" = "200" ]; then
    echo "✅ Health check passed!"
    exit 0
  fi
  
  echo "⏳ Waiting for service... (${elapsed}s / ${TIMEOUT}s)"
  sleep $INTERVAL
  elapsed=$((elapsed + INTERVAL))
done

echo "❌ Health check failed after ${TIMEOUT}s"
exit 1
```

Make executable:
```bash
chmod +x scripts/health-check.sh
```

---

## Verification

### Test CI Pipeline

1. Create a feature branch
2. Make a change and push
3. Create pull request
4. Verify CI runs all tests

### Test CD Pipeline

1. Merge to `develop` branch
2. Verify deployment to staging
3. Check staging environment
4. Merge to `main` branch
5. Verify deployment to production

---

## Troubleshooting

**Tests failing in CI but passing locally:**
- Check Node.js version matches
- Verify environment variables set
- Check service dependencies (DB, Redis)

**Docker build fails:**
- Review Dockerfile syntax
- Check Docker Hub credentials
- Verify base image availability

**Kubernetes deployment fails:**
- Check cluster connectivity
- Verify image pulled successfully
- Review pod logs: `kubectl logs -n production deployment/blog-engine`

---

## Related

- [Release Process](../../runbooks/release-process.md)
- [Deployment Guide](deploy-to-production.md)
- [Monitoring Setup](monitoring-setup.md)
