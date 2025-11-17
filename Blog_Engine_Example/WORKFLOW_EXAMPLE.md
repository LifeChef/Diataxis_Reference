# Complete Workflow: Business Request to Production Release

**Feature Request Example**: "We need a new way to list messages in replies to blog posts, they must be one after the other, with ability of users to sign thru SSO providers and leave a comment"

This document shows the complete end-to-end workflow of how teams collaborate, what documentation gets created at each stage, and where it lives in the Diataxis framework.

---

## 📋 Phase 1: Business Analysis & Discovery (Week 1)

### Team Members Involved
- **Product Manager** (Sarah) - Owns requirements
- **Business Analyst** (Mike) - Translates business to technical
- **Tech Lead** (Alex) - Technical feasibility

### Activities

#### 1.1 Requirements Gathering (Days 1-2)
**Owner**: Product Manager (Sarah)

**Actions**:
- Stakeholder interviews
- User story creation
- Acceptance criteria definition

**Documentation Created**:
```
📁 /features/comment-system/
  └── requirements.md          ← Created by PM
```

**Content Includes**:
- Business objectives
- User stories
- Success metrics (engagement rate, comment velocity)
- Acceptance criteria
- Non-functional requirements (performance, scalability)

**Diataxis Category**: N/A (Feature docs, not user-facing)

---

#### 1.2 Technical Analysis (Days 3-4)
**Owner**: Business Analyst (Mike) + Tech Lead (Alex)

**Actions**:
- Break down feature into components:
  - **Component 1**: Threaded comments system
  - **Component 2**: SSO authentication (Google, GitHub)
  - **Component 3**: Comment listing/pagination
- Identify dependencies (Auth Service, Post Service, Database)
- Risk assessment

**Documentation Created**:
```
📁 /features/comment-system/
  ├── requirements.md          (updated)
  └── design.md                ← Created by Tech Lead
```

**Handoff**: 
- **From**: PM (Sarah) → **To**: Tech Lead (Alex)
- **Format**: Requirements review meeting + Jira tickets
- **Acceptance**: Tech Lead signs off on feasibility

---

#### 1.3 Architecture Decision (Day 5)
**Owner**: Architect (David) + Tech Lead (Alex)

**Actions**:
- Decide on architecture approach
- Create Architecture Decision Record (ADR)
- Review with engineering team

**Documentation Created**:
```
📁 /docs/decisions/
  └── adr-006-comment-service.md    ← Created by Architect
```

**Content**:
- **Context**: Need for comment/reply system
- **Decision**: Microservice vs. extending Post Service
- **Rationale**: Pros/cons of each approach
- **Consequences**: Impact on system
- **Alternatives Considered**: Monolith approach, third-party service

**Diataxis Category**: **Explanation** (why we chose this approach)

**Handoff**:
- **From**: Architect (David) → **To**: Backend Team
- **Format**: Architecture review meeting
- **Acceptance**: ADR approved by CTO

---

## 💻 Phase 2: Development (Weeks 2-4)

### Team Members Involved
- **Backend Developer** (John) - API implementation
- **Frontend Developer** (Emma) - UI implementation
- **Database Engineer** (Lisa) - Schema design
- **Security Engineer** (Tom) - OAuth implementation

---

#### 2.1 Database Schema Design (Week 2, Days 1-2)
**Owner**: Database Engineer (Lisa)

**Actions**:
- Design `comments` table schema
- Define relationships (posts → comments → replies)
- Create migration scripts
- Update ER diagrams

**Documentation Created/Updated**:
```
📁 /reference/database/
  └── schema.md                ← Updated by DBA
```

**Content Added**:
```sql
CREATE TABLE comments (
  id UUID PRIMARY KEY,
  post_id UUID REFERENCES posts(id),
  user_id UUID REFERENCES users(id),
  parent_id UUID REFERENCES comments(id),  -- For replies
  content TEXT NOT NULL,
  created_at TIMESTAMP,
  updated_at TIMESTAMP,
  deleted_at TIMESTAMP
);

CREATE INDEX idx_comments_post ON comments(post_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_comments_parent ON comments(parent_id) WHERE deleted_at IS NULL;
```

**Diataxis Category**: **Reference** (technical specifications)

**Handoff**:
- **From**: DBA (Lisa) → **To**: Backend Dev (John)
- **Format**: Schema review + migration scripts
- **Acceptance**: Schema approved in code review

---

#### 2.2 SSO Authentication Implementation (Week 2, Days 3-5)
**Owner**: Security Engineer (Tom)

**Actions**:
- Implement OAuth2 flow for Google & GitHub
- Add SSO endpoints to Auth Service
- Configure OAuth credentials
- Security testing

**Documentation Created/Updated**:
```
📁 /docs/security/
  └── authentication.md        ← Updated by Security Engineer

📁 /reference/api/
  └── endpoints.md             ← Updated with new endpoints
```

**New API Endpoints**:
- `GET /auth/oauth/google`
- `GET /auth/oauth/github`
- `GET /auth/oauth/:provider/callback`

**Diataxis Categories**: 
- `/docs/security/` → **Explanation** (how SSO works, security model)
- `/reference/api/` → **Reference** (API specs)

**Handoff**:
- **From**: Security (Tom) → **To**: Backend Dev (John)
- **Format**: Auth library + example code
- **Acceptance**: Security review passed

---

#### 2.3 Comment Service API Development (Week 3)
**Owner**: Backend Developer (John)

**Actions**:
- Implement Comment Service microservice
- Create API endpoints:
  - `GET /posts/:postId/comments` - List comments
  - `POST /posts/:postId/comments` - Create comment
  - `POST /comments/:id/reply` - Reply to comment
  - `DELETE /comments/:id` - Delete comment
- Add pagination, sorting (newest, oldest, threaded)
- Integrate with Auth Service for SSO
- Write unit tests (80% coverage minimum)

**Documentation Created/Updated**:
```
📁 /reference/api/
  └── endpoints.md             ← Updated by Backend Dev

📁 /docs/architecture/services/
  └── README.md                ← Updated with Comment Service

📁 /features/comment-system/
  ├── design.md                ← Updated with implementation details
  └── testing.md               ← Created by Backend Dev
```

**Diataxis Categories**:
- `/reference/api/` → **Reference** (API documentation)
- `/docs/architecture/` → **Reference** (system architecture)
- `/features/*/testing.md` → Internal QA doc

**Handoff**:
- **From**: Backend (John) → **To**: Frontend (Emma) + QA (Rachel)
- **Format**: API documentation + Swagger/OpenAPI spec
- **Acceptance**: API endpoints deployed to staging

---

#### 2.4 Frontend Implementation (Week 3-4)
**Owner**: Frontend Developer (Emma)

**Actions**:
- Build comment UI components
- Implement SSO login buttons
- Add threaded comment display (nested replies)
- Pagination controls
- Real-time updates (WebSocket or polling)
- Integration tests

**Documentation Created** (minimal - mostly internal):
```
📁 /features/comment-system/
  └── design.md                ← Updated with UI mockups
```

**Handoff**:
- **From**: Frontend (Emma) → **To**: QA (Rachel)
- **Format**: Feature branch deployed to staging environment
- **Acceptance**: UI matches design specs

---

## 🧪 Phase 3: Testing & Documentation (Week 4-5)

### Team Members Involved
- **QA Engineer** (Rachel) - Test execution
- **Technical Writer** (Carlos) - User documentation
- **Backend Dev** (John) - Bug fixes

---

#### 3.1 QA Testing (Week 4)
**Owner**: QA Engineer (Rachel)

**Actions**:
- Functional testing (all user flows)
- SSO testing (Google, GitHub)
- Performance testing (load tests)
- Security testing (XSS, CSRF, auth bypass)
- Regression testing
- Create test cases

**Documentation Created/Updated**:
```
📁 /features/comment-system/
  └── testing.md               ← Updated by QA
```

**Bug Tracking**:
- 12 bugs found (3 critical, 5 medium, 4 low)
- All critical bugs must be fixed before release

**Handoff**:
- **From**: QA (Rachel) → **To**: Backend Dev (John) for fixes
- **Format**: Bug tickets in Jira with reproduction steps
- **Acceptance**: All critical/medium bugs resolved

---

#### 3.2 User Documentation (Week 5, Days 1-3)
**Owner**: Technical Writer (Carlos)

**Actions**:
- Write end-user documentation across all Diataxis categories
- Create screenshots/GIFs
- Review with PM for accuracy

**Documentation Created**:

##### 3.2.1 Tutorial (Learning-Oriented)
```
📁 /tutorials/getting-started/
  └── 03-your-first-comment.md     ← Created by Tech Writer
```

**Content**:
```markdown
# Your First Comment on a Blog Post

Learn how to sign in and leave your first comment on our platform.

## Prerequisites
- A Google or GitHub account

## Step 1: Sign in with SSO
1. Click "Sign in with Google" on the homepage
2. Approve the permissions...
[Complete step-by-step walkthrough]

## Step 2: Find a Blog Post
...

## Step 3: Leave a Comment
...

## Expected Result
You should see your comment appear immediately below the post.
```

**Diataxis Category**: **Tutorial** → `/tutorials/`

---

##### 3.2.2 How-To Guide (Problem-Oriented)
```
📁 /how-to/interaction/
  └── reply-to-comments.md         ← Created by Tech Writer
```

**Content**:
```markdown
# How to Reply to a Comment

**Problem**: You want to respond to someone's comment on a blog post.

## Prerequisites
- You must be signed in

## Solution

### Quick Steps
1. Find the comment you want to reply to
2. Click the "Reply" button below it
3. Type your response in the text box
4. Click "Post Reply"

Your reply will appear nested under the original comment.

## Troubleshooting
- **Can't see Reply button?** → You need to sign in first
- **Reply not appearing?** → Refresh the page
...
```

**Diataxis Category**: **How-To Guide** → `/how-to/`

---

##### 3.2.3 Reference Documentation (Information-Oriented)
```
📁 /reference/api/
  └── endpoints.md                 ← Updated by Tech Writer
```

**Content**:
```markdown
### POST /posts/:postId/comments

Create a new comment on a post.

**Authentication**: Required (JWT or OAuth token)

**Request Body**:
{
  "content": "string (required, 1-2000 chars)",
  "parentId": "uuid (optional, for replies)"
}

**Response** (201):
{
  "id": "uuid",
  "content": "Great post!",
  "author": {...},
  "createdAt": "2024-11-05T10:00:00Z"
}

**Errors**:
- 401: Not authenticated
- 403: Forbidden (banned user)
- 404: Post not found
- 422: Validation error
```

**Diataxis Category**: **Reference** → `/reference/`

---

##### 3.2.4 Explanation (Understanding-Oriented)
```
📁 /explanation/concepts/
  └── threaded-comments.md         ← Created by Tech Writer
```

**Content**:
```markdown
# Understanding Threaded Comments

## What are Threaded Comments?

Threaded comments allow users to reply directly to specific comments,
creating nested conversation threads rather than a flat list.

## Why Use Threading?

**Benefits**:
- Clearer conversations
- Better context for replies
- Improved engagement

## How It Works

[Diagram showing parent-child relationships]

When you reply to a comment, your reply is linked via `parent_id`...

## Design Decisions

We chose a single-level threading model (replies to replies are not
nested further) because...
```

**Diataxis Category**: **Explanation** → `/explanation/`

---

**Handoff**:
- **From**: Tech Writer (Carlos) → **To**: PM (Sarah) for review
- **Format**: Documentation review meeting
- **Acceptance**: PM approves all user-facing docs

---

## 🚀 Phase 4: Deployment Preparation (Week 5, Days 4-5)

### Team Members Involved
- **DevOps Engineer** (Chris) - Deployment
- **Backend Developer** (John) - Production monitoring
- **Tech Writer** (Carlos) - Runbooks

---

#### 4.1 Deployment Runbook Creation
**Owner**: DevOps Engineer (Chris)

**Actions**:
- Create deployment checklist
- Write rollback procedures
- Configure monitoring alerts
- Set up feature flags

**Documentation Created**:
```
📁 /runbooks/
  ├── deployment.md            ← Updated by DevOps
  └── incident-response.md     ← Updated with new scenarios
```

**Content**:
```markdown
## Deploying Comment Service v1.0

### Pre-deployment Checklist
- [ ] Database migrations tested in staging
- [ ] OAuth credentials configured in prod
- [ ] Rate limiting configured (100 comments/hour/user)
- [ ] Monitoring dashboards created
- [ ] Feature flag `comments_enabled` = false

### Deployment Steps
1. Apply database migrations
2. Deploy Comment Service (blue-green deployment)
3. Verify health checks
4. Enable feature flag for 5% of users
...

### Rollback Procedure
If errors > 5% or latency > 500ms:
1. Disable feature flag immediately
2. Roll back Comment Service
3. Investigate root cause
```

**Diataxis Category**: **How-To Guide** (operational) → `/runbooks/`

---

#### 4.2 Monitoring & Alerting Setup
**Owner**: DevOps Engineer (Chris)

**Actions**:
- Create Grafana dashboards
- Configure alerts (error rate, latency)
- Set up log aggregation
- Document SLIs/SLOs

**Documentation Updated**:
```
📁 /runbooks/
  └── monitoring.md            ← Updated by DevOps
```

**Content**:
```markdown
## Comment Service Monitoring

### Key Metrics
- **Comment Creation Rate**: Target 100/min
- **API Latency (P95)**: < 200ms
- **Error Rate**: < 1%
- **Database Connection Pool**: < 80% utilization

### Alerts
- **High Error Rate**: > 5% errors for 5 minutes → Page on-call
- **High Latency**: P95 > 500ms → Slack alert
...
```

**Diataxis Category**: **Reference** (monitoring specs) → `/runbooks/`

---

## 🎉 Phase 5: Release & Post-Launch (Week 6)

### Team Members Involved
- **Product Manager** (Sarah) - Release coordination
- **DevOps Engineer** (Chris) - Deployment execution
- **Support Team** (Jennifer) - User support
- **All Team** - Monitoring

---

#### 5.1 Phased Rollout (Days 1-3)
**Owner**: DevOps Engineer (Chris) + PM (Sarah)

**Actions**:
- **Day 1**: Enable for 5% of users
- **Day 2**: Monitor metrics, increase to 25%
- **Day 3**: Full rollout to 100%

**Rollout Plan**:
```
Week 6, Day 1 (5% users):
- Monitor: Error rate, latency, user feedback
- Success Criteria: < 1% errors, P95 < 200ms
- Rollback Trigger: > 5% errors OR user complaints > 10

Week 6, Day 2 (25% users):
- Continued monitoring
- Support team trained on new feature

Week 6, Day 3 (100% users):
- Full rollout
- Announcement blog post published
```

---

#### 5.2 Support Documentation
**Owner**: Support Team Lead (Jennifer) + Tech Writer (Carlos)

**Actions**:
- Update internal knowledge base
- Create support macros for common issues
- Train support team

**Documentation Created**:
```
📁 /how-to/troubleshooting/
  └── comment-issues.md        ← Created by Support Team
```

**Content**:
```markdown
# Troubleshooting Comment Issues

## User can't see comment button
**Cause**: Not signed in
**Solution**: Direct user to /auth/login

## SSO login fails
**Cause**: OAuth misconfiguration
**Solution**: Check OAuth credentials, redirect URI
...
```

**Diataxis Category**: **How-To Guide** → `/how-to/troubleshooting/`

---

#### 5.3 Post-Launch Monitoring (Week 6+)
**Owner**: Backend Developer (John) + DevOps (Chris)

**Actions**:
- Daily metric review for first week
- Bug triage and fixes
- Performance optimization
- Gather user feedback

**Documentation Updated**:
```
📁 /features/comment-system/
  └── requirements.md          ← Updated with actual metrics
```

**Success Metrics Achieved** (Week 7 data):
- ✅ 1,200 comments/day (exceeded 1,000 target)
- ✅ 85% of users use SSO (exceeded 70% target)
- ✅ P95 latency: 145ms (target: < 200ms)
- ✅ Error rate: 0.3% (target: < 1%)

---

## 📊 Complete Documentation Map

Here's where ALL documentation lives in the Diataxis framework:

```
Blog_Engine_Example/
│
├── features/comment-system/          [FEATURE DOCS - Internal]
│   ├── requirements.md               (PM writes, BA updates)
│   ├── design.md                     (Tech Lead writes)
│   └── testing.md                    (QA writes)
│
├── docs/                             [SYSTEM-WIDE DOCS]
│   ├── decisions/
│   │   └── adr-006-comment-service.md    [EXPLANATION]
│   ├── security/
│   │   └── authentication.md             [EXPLANATION + REFERENCE]
│   └── architecture/services/
│       └── README.md                     [REFERENCE]
│
├── tutorials/                        [TUTORIALS - Learning]
│   └── getting-started/
│       └── 03-your-first-comment.md      (Tech Writer)
│
├── how-to/                           [HOW-TO - Problem-solving]
│   ├── interaction/
│   │   └── reply-to-comments.md          (Tech Writer)
│   └── troubleshooting/
│       └── comment-issues.md             (Support Team)
│
├── reference/                        [REFERENCE - Information]
│   ├── api/
│   │   └── endpoints.md                  (Backend Dev + Tech Writer)
│   └── database/
│       └── schema.md                     (DBA)
│
├── explanation/                      [EXPLANATION - Understanding]
│   └── concepts/
│       └── threaded-comments.md          (Tech Writer + Architect)
│
└── runbooks/                         [OPERATIONAL - How-To]
    ├── deployment.md                     (DevOps)
    ├── monitoring.md                     (DevOps)
    └── incident-response.md              (DevOps + On-call)
```

---

## 🔄 Handoff Summary

| Phase | From → To | Artifact | Acceptance Criteria |
|-------|-----------|----------|---------------------|
| **1. Discovery** | PM → Tech Lead | `requirements.md` | Feasibility signed off |
| **2. Architecture** | Tech Lead → Architect | Feature spec | ADR approved |
| **3. Design** | Architect → Backend | `adr-006-*.md` | Architecture review passed |
| **4. Database** | DBA → Backend | `schema.md` + migrations | Schema approved |
| **5. Backend API** | Backend → Frontend + QA | `endpoints.md` | API deployed to staging |
| **6. Frontend** | Frontend → QA | Staging deployment | UI matches design |
| **7. QA** | QA → Backend | Bug reports | All critical bugs fixed |
| **8. Documentation** | Tech Writer → PM | User docs (all 4 types) | Docs approved |
| **9. Deployment** | DevOps → PM | `deployment.md` | Runbook validated |
| **10. Release** | PM → Support | Training materials | Support team trained |

---

## 👥 Roles & Responsibilities Summary

| Role | Primary Docs Created | Diataxis Categories | Tools Used |
|------|---------------------|---------------------|------------|
| **Product Manager** | `features/*/requirements.md` | N/A (internal) | Jira, Confluence |
| **Business Analyst** | `features/*/requirements.md` | N/A (internal) | Jira, Miro |
| **Architect** | `docs/decisions/adr-*.md` | **Explanation** | Draw.io, Markdown |
| **Tech Lead** | `features/*/design.md` | N/A (internal) | Markdown, Git |
| **Backend Developer** | `reference/api/endpoints.md` | **Reference** | Swagger, Postman |
| **Database Engineer** | `reference/database/schema.md` | **Reference** | SQL, dbdiagram.io |
| **Security Engineer** | `docs/security/*.md` | **Explanation** | Security tools |
| **Frontend Developer** | UI mockups (in `design.md`) | N/A | Figma, React |
| **QA Engineer** | `features/*/testing.md` | N/A (internal) | Jest, Cypress |
| **Technical Writer** | All user-facing docs | **All 4 categories** | Markdown, Git |
| **DevOps Engineer** | `runbooks/*.md` | **How-To** | Kubernetes, Terraform |
| **Support Team** | `how-to/troubleshooting/*.md` | **How-To** | Zendesk, Docs |

---

## 📈 Timeline Summary

| Week | Milestone | Team | Documentation |
|------|-----------|------|---------------|
| **1** | Requirements & Architecture | PM, BA, Architect | `requirements.md`, `adr-*.md` |
| **2-3** | Development | Backend, Frontend, DBA | `schema.md`, `endpoints.md`, `design.md` |
| **4** | QA Testing | QA | `testing.md` |
| **5** | User Docs + Deployment Prep | Tech Writer, DevOps | All 4 Diataxis categories, runbooks |
| **6** | Production Release | All | Post-launch metrics |

**Total Time**: 6 weeks from business request to public release

---

## 💡 Key Insights

### 1. **Documentation is Created Throughout SDLC**
- Not just at the end
- Each phase produces specific docs
- Living documentation (updated as code changes)

### 2. **Different Docs for Different Audiences**
- **Internal**: `/features/` (team only)
- **Developers**: `/reference/`, `/docs/` (technical)
- **Users**: `/tutorials/`, `/how-to/` (non-technical)
- **Operations**: `/runbooks/` (DevOps)

### 3. **Handoffs are Documentation-Driven**
- Each handoff includes updated docs
- Clear acceptance criteria
- No "tribal knowledge" - everything written down

### 4. **Diataxis Organizes User-Facing Docs**
- **Tutorials**: Onboarding new users
- **How-To**: Solving specific problems
- **Reference**: Looking up technical details
- **Explanation**: Understanding architecture/decisions

### 5. **Multiple People Touch Same Docs**
- Backend Dev writes initial API reference
- Tech Writer polishes for users
- Support Team adds troubleshooting
- Docs are collaborative, not siloed

---

## 🎯 Success Factors

✅ **Clear ownership** - Every doc has an owner  
✅ **Defined handoffs** - No ambiguity about next steps  
✅ **Acceptance criteria** - Clear "done" definition  
✅ **Living documentation** - Updated throughout lifecycle  
✅ **User-focused** - Diataxis ensures docs match user needs  
✅ **Operational readiness** - Runbooks before deployment  
✅ **Phased rollout** - Reduces risk, enables monitoring

---

## 📋 Quick Reference: Who Does What When

### Week 1: Discovery
- **Day 1-2**: PM writes requirements
- **Day 3-4**: BA + Tech Lead break down into components
- **Day 5**: Architect creates ADR

### Week 2-3: Development
- **Week 2, Day 1-2**: DBA designs schema → updates `/reference/database/schema.md`
- **Week 2, Day 3-5**: Security implements SSO → updates `/docs/security/authentication.md`
- **Week 3**: Backend builds API → updates `/reference/api/endpoints.md`
- **Week 3-4**: Frontend builds UI → minimal docs

### Week 4: Testing
- **QA**: Tests everything, logs bugs

### Week 5: Documentation & Deployment Prep
- **Day 1-3**: Tech Writer creates:
  - Tutorial: `/tutorials/getting-started/03-your-first-comment.md`
  - How-To: `/how-to/interaction/reply-to-comments.md`
  - Reference: Updates API docs
  - Explanation: `/explanation/concepts/threaded-comments.md`
- **Day 4-5**: DevOps creates deployment runbook

### Week 6: Release
- **Day 1**: 5% rollout
- **Day 2**: 25% rollout
- **Day 3**: 100% rollout
- **Ongoing**: Support Team creates troubleshooting docs

---

**Related Documentation**:
- [Contributing Guide](./CONTRIBUTING.md) - Development standards
- [Project Overview](./PROJECT_OVERVIEW.md) - System architecture
- [Roadmap](./ROADMAP.md) - Feature pipeline
- [Getting Started](./GETTING_STARTED.md) - Quick onboarding

**Questions?** Contact the Documentation Team or open an issue.

---

*This workflow represents the ideal process for the Blog Engine project. Adapt as needed for your team's context.*
