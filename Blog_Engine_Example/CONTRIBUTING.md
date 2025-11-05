# Contributing to Blog Engine

Thank you for contributing! This guide helps you contribute effectively to code and documentation.

## Quick Start

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Make your changes
4. Run tests: `npm test`
5. Commit: `git commit -m "feat: your feature description"`
6. Push: `git push origin feature/your-feature`
7. Create a Pull Request

---

## Code Standards

### Style Guide
- **JavaScript/TypeScript**: ESLint + Prettier configuration
- **Naming**: camelCase for variables, PascalCase for components/classes
- **Comments**: JSDoc for public APIs
- **Commits**: Follow [Conventional Commits](https://www.conventionalcommits.org/)

### Commit Message Format

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types**: feat, fix, docs, style, refactor, test, chore

**Examples**:
```
feat(post-service): add draft post functionality
fix(auth): resolve token expiration bug
docs(api): update endpoint documentation
```

---

## Development Workflow

### 1. Pick a Task
- Check GitHub Issues or project board
- Assign yourself to avoid duplicate work
- Ask questions if requirements unclear

### 2. Create Branch
```bash
git checkout main
git pull origin main
git checkout -b feature/issue-123-description
```

### 3. Develop Locally
```bash
# Start services
docker-compose up -d

# Run in development mode
npm run dev

# Watch tests
npm run test:watch
```

### 4. Test Your Changes

**Required before PR**:
```bash
# Unit tests
npm test

# Integration tests
npm run test:integration

# Linting
npm run lint

# Type checking
npm run type-check
```

### 5. Create Pull Request

**PR Title**: Follow commit message format  
**PR Description** Must include:
- What changed and why
- How to test
- Screenshots (if UI change)
- Related issue number

**PR Template**:
```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing completed

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] No new warnings
```

---

## Testing Requirements

### Unit Tests
- **Coverage**: Minimum 80% for new code
- **Framework**: Jest
- **Location**: `__tests__` folders next to source

```javascript
describe('PostService', () => {
  it('should create a new post', async () => {
    const post = await postService.create({title: 'Test', content: 'Content'});
    expect(post.id).toBeDefined();
  });
});
```

### Integration Tests
- Test API endpoints end-to-end
- Use test database
- Clean up after tests

```javascript
describe('POST /api/posts', () => {
  it('should create post with valid data', async () => {
    const response = await request(app)
      .post('/api/posts')
      .send({title: 'Test', content: 'Content'})
      .expect(201);
    
    expect(response.body.id).toBeDefined();
  });
});
```

### E2E Tests (Critical Paths Only)
- User flows: signup, login, create post, comment
- Framework: Cypress (web), Detox (mobile)

---

## Code Review Process

### For Authors
1. Ensure CI passes before requesting review
2. Respond to feedback constructively
3. Request re-review after changes
4. Don't merge your own PRs

### For Reviewers
1. Review within 24 hours
2. Be constructive and specific
3. Approve only if you'd ship it
4. Check: correctness, tests, style, security

### Review Checklist
- [ ] Code solves the stated problem
- [ ] Tests are adequate
- [ ] No security vulnerabilities
- [ ] Performance acceptable
- [ ] Documentation updated
- [ ] Follows style guide
- [ ] No unnecessary complexity

---

## Documentation Standards

### When to Update Docs
- New features or APIs
- Changed behavior
- New configuration options
- Architectural decisions

### Documentation Types

**Tutorials** (Learning-oriented):
- Step-by-step, learning by doing
- Complete working examples
- Beginner-friendly

**How-To Guides** (Problem-oriented):
- Solve specific problems
- Assume some knowledge
- Direct and concise

**Reference** (Information-oriented):
- API specs, configuration
- Accurate and complete
- Neutral tone

**Explanations** (Understanding-oriented):
- Concepts and decisions
- Why, not how
- Context and background

See `/Diataxis_Templates` for templates and examples.

---

## Security

### Reporting Vulnerabilities
**Do NOT** create public issues for security issues.

Email: security@blogengine.com

Include:
- Description of vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if any)

### Security Best Practices
- Never commit secrets (API keys, passwords)
- Use environment variables
- Validate all inputs
- Sanitize outputs
- Follow OWASP guidelines
- Keep dependencies updated

---

## Database Migrations

### Creating Migrations
```bash
npm run migrate:create -- add_email_to_users
```

### Migration Format
```javascript
exports.up = async (knex) => {
  await knex.schema.table('users', (table) => {
    table.string('email').notNullable().unique();
  });
};

exports.down = async (knex) => {
  await knex.schema.table('users', (table) => {
    table.dropColumn('email');
  });
};
```

### Rules
- Always provide `up` and `down`
- Test rollback before merging
- Never modify existing migrations
- Include data migrations if needed

---

## Release Process

### Versioning
We use Semantic Versioning (SemVer):
- MAJOR: Breaking changes
- MINOR: New features (backward compatible)
- PATCH: Bug fixes

### Release Cycle
- **Weekly**: Minor releases (features)
- **As Needed**: Patch releases (bugs)
- **Quarterly**: Major releases (breaking changes)

### Release Checklist
- [ ] All tests passing
- [ ] Changelog updated
- [ ] Version bumped
- [ ] Migration notes (if breaking)
- [ ] Docs updated
- [ ] Stakeholders notified

See [Release Process](runbooks/release-process.md) for details.

---

## Getting Help

### Questions
- **Code Questions**: #blog-engine-dev on Slack
- **Process Questions**: Ask your team lead
- **Bugs**: Create GitHub issue

### Resources
- [Getting Started Guide](GETTING_STARTED.md)
- [API Reference](reference/api/endpoints.md)
- [System Architecture](reference/architecture/system-architecture.md)

---

## Recognition

Contributors are recognized in:
- CONTRIBUTORS.md file
- Release notes
- Monthly team meetings

Top contributors may be invited to:
- Architecture review meetings
- Tech lead rotation
- Conference speaking opportunities

---

**Thank you for contributing to Blog Engine! 🎉**
