# Test Strategy

> **Document Type**: Runbook (SAMPLE)  
> **Audience**: QA Engineers, Developers, Engineering Managers  
> **Last Updated**: November 5, 2024

---

## Overview

This document defines the testing strategy for the Blog Engine, including test types, coverage goals, automation approach, and quality gates.

---

## Testing Pyramid

```
              /\
             /E2E\      10% - End-to-End Tests
            /------\
           /  INT   \   20% - Integration Tests
          /----------\
         /    UNIT    \ 70% - Unit Tests
        /--------------\
```

### Distribution
- **70% Unit Tests**: Fast, isolated, developer-written
- **20% Integration Tests**: API contracts, service interactions
- **10% E2E Tests**: Critical user flows only

---

## Test Types

### 1. Unit Tests

**Purpose**: Test individual functions/components in isolation

**Framework**: Jest + React Testing Library

**Coverage Goal**: 80% minimum

**Examples**:
```javascript
// Backend - Service logic
describe('PostService', () => {
  it('should create a post with valid data', async () => {
    const postData = { title: 'Test', content: 'Content' };
    const result = await postService.create(postData);
    expect(result).toHaveProperty('id');
  });
});

// Frontend - Component
describe('PostCard', () => {
  it('should render post title and excerpt', () => {
    const post = { title: 'Test', excerpt: 'Excerpt' };
    render(<PostCard post={post} />);
    expect(screen.getByText('Test')).toBeInTheDocument();
  });
});
```

**Run Command**:
```bash
npm run test              # Run all unit tests
npm run test:watch        # Watch mode
npm run test:coverage     # Generate coverage report
```

---

### 2. Integration Tests

**Purpose**: Test API endpoints and service interactions

**Framework**: Supertest (API), Testcontainers (DB)

**Coverage Goal**: All critical API endpoints

**Examples**:
```javascript
describe('POST /api/posts', () => {
  it('should create post with authentication', async () => {
    const token = await getAuthToken();
    const response = await request(app)
      .post('/api/posts')
      .set('Authorization', `Bearer ${token}`)
      .send({ title: 'Test', content: 'Content' })
      .expect(201);
    
    expect(response.body.id).toBeDefined();
  });

  it('should reject unauthenticated requests', async () => {
    await request(app)
      .post('/api/posts')
      .send({ title: 'Test', content: 'Content' })
      .expect(401);
  });
});
```

**Run Command**:
```bash
npm run test:integration
```

---

### 3. End-to-End Tests

**Purpose**: Test complete user flows across systems

**Framework**: Cypress (Web), Detox (Mobile)

**Coverage**: Critical paths only

**Critical User Flows**:
1. User signup → email verification → first post
2. Login → create post → publish → view
3. Create post → add comment → receive notification
4. Admin login → moderate content → approve post

**Example**:
```javascript
// Cypress - Web E2E
describe('Post Creation Flow', () => {
  it('should allow user to create and publish post', () => {
    cy.login('user@example.com', 'password');
    cy.visit('/posts/new');
    cy.get('[data-testid="title-input"]').type('My Post');
    cy.get('[data-testid="content-editor"]').type('Post content');
    cy.get('[data-testid="publish-btn"]').click();
    cy.url().should('include', '/posts/');
    cy.contains('My Post').should('be.visible');
  });
});
```

**Run Command**:
```bash
npm run test:e2e          # Headless mode
npm run test:e2e:open     # Interactive mode
```

---

### 4. Contract Tests

**Purpose**: Ensure service APIs match contracts

**Framework**: Pact

**Coverage**: All inter-service communication

**Example**:
```javascript
// Consumer test (Frontend expects API contract)
describe('Post API Contract', () => {
  it('should return post with expected schema', () => {
    return provider
      .addInteraction({
        state: 'post exists',
        uponReceiving: 'a request for a post',
        withRequest: { method: 'GET', path: '/api/posts/1' },
        willRespondWith: {
          status: 200,
          body: Matchers.like({
            id: 1,
            title: 'Post Title',
            content: 'Post content',
            authorId: 123
          })
        }
      })
      .then(() => fetchPost(1))
      .then(post => expect(post.title).toBe('Post Title'));
  });
});
```

---

### 5. Performance Tests

**Purpose**: Verify system performance under load

**Framework**: k6, Apache JMeter

**Metrics**:
- Response time: < 200ms (P95)
- Throughput: 1000 req/sec
- Error rate: < 1%

**Example**:
```javascript
// k6 load test
import http from 'k6/http';
import { check } from 'k6';

export let options = {
  vus: 100,          // 100 virtual users
  duration: '5m',    // 5 minutes
};

export default function () {
  let response = http.get('https://api.blogengine.com/posts');
  check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 200ms': (r) => r.timings.duration < 200,
  });
}
```

**Run Command**:
```bash
npm run test:performance
```

---

### 6. Security Tests

**Purpose**: Identify security vulnerabilities

**Tools**:
- **OWASP ZAP**: Automated security scanning
- **npm audit**: Dependency vulnerabilities
- **Snyk**: Container and dependency scanning

**Test Areas**:
- SQL Injection
- XSS (Cross-Site Scripting)
- CSRF (Cross-Site Request Forgery)
- Authentication bypass
- Authorization flaws

**Run Command**:
```bash
npm audit                    # Dependency vulnerabilities
npm run test:security        # OWASP ZAP scan
```

---

## Test Environments

### Local Development
- **Purpose**: Developer testing
- **Database**: SQLite or Docker PostgreSQL
- **Setup**: `docker-compose up`

### CI/CD (GitHub Actions)
- **Purpose**: Automated testing on every PR
- **Database**: Postgres container
- **Tests Run**: Unit, Integration, Linting

### Staging
- **Purpose**: Pre-production testing
- **Database**: Production-like Postgres
- **Tests Run**: E2E, Performance, Security

### Production
- **Purpose**: Smoke tests only
- **Tests Run**: Critical path validation
- **Frequency**: After every deployment

---

## Quality Gates

### Pull Request Requirements
- [ ] All unit tests pass
- [ ] Code coverage ≥ 80%
- [ ] No linting errors
- [ ] No security vulnerabilities
- [ ] 1+ code review approval

### Pre-Merge to Main
- [ ] All integration tests pass
- [ ] Contract tests pass
- [ ] No breaking API changes (or documented)

### Pre-Deployment
- [ ] All E2E tests pass on staging
- [ ] Performance benchmarks met
- [ ] Security scan passed (no critical/high issues)
- [ ] Database migrations tested

---

## Test Data Management

### Test Data Strategy
- **Fixtures**: Predefined test data in `/tests/fixtures`
- **Factories**: Generate test data programmatically
- **Seeders**: Database seeders for integration tests

### Example Factory
```javascript
// factories/post.factory.js
const createPost = (overrides = {}) => ({
  title: 'Test Post',
  content: 'Test content',
  authorId: 1,
  status: 'published',
  ...overrides
});

// Usage
const draftPost = createPost({ status: 'draft' });
```

### Database Seeding
```bash
# Seed test database
npm run db:seed:test

# Reset test database
npm run db:reset:test
```

---

## Continuous Integration

### GitHub Actions Workflow
```yaml
name: Test Suite
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Setup Node
        uses: actions/setup-node@v2
        with:
          node-version: '18'
      - name: Install dependencies
        run: npm ci
      - name: Run unit tests
        run: npm test
      - name: Run integration tests
        run: npm run test:integration
      - name: Upload coverage
        uses: codecov/codecov-action@v2
```

---

## Test Metrics & Reporting

### Tracked Metrics
- **Test Coverage**: Target 80%, current 72%
- **Test Execution Time**: Target < 5 min for CI
- **Flaky Test Rate**: Target < 2%
- **Test-to-Code Ratio**: Target 1:1 (lines of test vs code)

### Reporting
- **Coverage Reports**: CodeCov (automatic on PR)
- **Test Results**: GitHub Actions summary
- **Performance**: k6 HTML reports
- **Security**: Snyk dashboard

---

## Test Maintenance

### Weekly
- [ ] Review flaky tests
- [ ] Update test data fixtures
- [ ] Check coverage trends

### Monthly
- [ ] Review and update E2E scenarios
- [ ] Performance baseline validation
- [ ] Security test updates

### Quarterly
- [ ] Full test strategy review
- [ ] Framework/tool updates
- [ ] Test suite optimization

---

## Best Practices

### DO
✅ Write tests alongside code (TDD encouraged)  
✅ Use descriptive test names  
✅ Keep tests independent and isolated  
✅ Mock external dependencies  
✅ Use test factories for data generation  
✅ Run tests locally before pushing

### DON'T
❌ Commit failing tests  
❌ Skip tests to "fix later"  
❌ Test implementation details  
❌ Share state between tests  
❌ Use production data in tests  
❌ Ignore flaky tests

---

## Troubleshooting

### Common Issues

**Tests fail locally but pass in CI**
- Check Node version mismatch
- Ensure all dependencies installed
- Clear test database and re-seed

**Flaky E2E tests**
- Add explicit waits instead of fixed delays
- Check for race conditions
- Verify test data isolation

**Slow test suite**
- Parallelize test execution
- Use test.only during development
- Review database setup/teardown

---

## Related Documentation

- [CONTRIBUTING.md](../CONTRIBUTING.md)
- [CI/CD Setup](../how-to/deployment/cicd-setup.md)
- [Quality Gates](quality-gates.md)

---

**Maintained By**: QA Team + Tech Lead  
**Review Frequency**: Quarterly  
**Next Review**: February 5, 2025
