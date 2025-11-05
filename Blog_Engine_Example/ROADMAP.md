# Blog Engine - Project Roadmap

> **Current Phase**: Development - MVP  
> **Sprint**: 12 of 16  
> **Target Launch**: December 15, 2024  
> **Last Updated**: November 5, 2024

---

## Overview

This roadmap tracks high-level milestones, phases, and progress. For detailed feature tasks, see `/features/`.

---

## Release Timeline

```
Nov 2025          Dec 2025          Jan 2026          Feb 2026
    |                 |                 |                 |
    ├─ Sprint 12      ├─ Sprint 16      ├─ Sprint 20      ├─ Sprint 24
    │  (Current)      │  MVP Launch     │  Beta Release   │  GA Release
    │                 │                 │                 │
    └─ Core Platform  └─ User Features  └─ Performance    └─ Scale & Polish
       70% complete      Planned           Planned           Planned
```

---

## Phase 1: Core Platform (Current - 70% Complete)

**Goal**: Build foundational services and infrastructure  
**Target**: Sprint 12 (This Sprint)  
**Status**: 🟡 On Track (minor delays)

### Milestones

| Milestone | Status | Owner | Completion |
|-----------|--------|-------|------------|
| Architecture Design | ✅ Done | Architecture Team | 100% |
| Infrastructure Setup | ✅ Done | DevOps | 100% |
| Database Schema | ✅ Done | Backend | 100% |
| Auth Service | ✅ Done | Backend | 100% |
| Post Service | 🔄 In Progress | Backend | 60% |
| Comment Service | 📋 Planned | Backend | 0% |
| API Gateway | 🔄 In Progress | Backend | 80% |

**Blockers**:
1. Post service performance (large image uploads) - Investigating CDN solution
2. Database migration tooling - Decision needed on Flyway vs Liquibase

**Deliverables**:
- ✅ All core services containerized
- ✅ CI/CD pipeline operational
- 🔄 Basic CRUD operations for posts
- 📋 Comment system (next sprint)

**Feature Details**: See `/features/user-authentication/`, `/features/blog-posts/`

---

## Phase 2: User Features (Upcoming - Sprint 13-16)

**Goal**: Complete user-facing functionality  
**Target**: Sprint 16 (MVP Launch)  
**Status**: 📋 Planned

### Milestones

| Milestone | Status | Owner | Target Sprint |
|-----------|--------|-------|---------------|
| Email Notifications | 🔄 Started | Backend | Sprint 13 |
| Push Notifications | 📋 Planned | Mobile | Sprint 14 |
| User Profiles | 📋 Planned | Backend + Frontend | Sprint 13 |
| Follow/Unfollow | 📋 Planned | Backend | Sprint 14 |
| Admin Panel | 📋 Planned | Frontend | Sprint 15-16 |
| Content Moderation | 📋 Planned | Backend + Admin | Sprint 16 |

**Dependencies**:
- Push notifications require mobile apps (Sprint 14)
- Admin panel needs role-based access control (Sprint 15)

**Deliverables**:
- Complete notification system (email + push)
- User interaction features (follow, like)
- Basic admin panel for content management

**Feature Details**: See `/features/notifications/`, `/features/admin-panel/`

---

## Phase 3: Performance & Scale (Sprint 17-20)

**Goal**: Optimize for production load  
**Target**: Sprint 20 (Beta Launch)  
**Status**: 📋 Planned

### Milestones

| Milestone | Status | Owner | Target Sprint |
|-----------|--------|-------|---------------|
| Load Testing | 📋 Planned | QA + DevOps | Sprint 17 |
| Caching Strategy | 📋 Planned | Backend | Sprint 17-18 |
| Database Optimization | 📋 Planned | Backend + DBA | Sprint 18 |
| CDN Integration | 📋 Planned | DevOps | Sprint 18 |
| Search Optimization | 📋 Planned | Backend | Sprint 19 |
| Monitoring Setup | 📋 Planned | DevOps | Sprint 19-20 |

**Target Metrics**:
- 1,000 concurrent users
- < 200ms API response time (p95)
- 99.9% uptime
- < 2s page load time

**Deliverables**:
- Performance benchmarks documented
- Caching implemented (Redis)
- Monitoring dashboards (Prometheus + Grafana)
- Load testing results

---

## Phase 4: Production Readiness (Sprint 21-24)

**Goal**: Launch-ready system  
**Target**: Sprint 24 (GA Release)  
**Status**: 📋 Planned

### Milestones

| Milestone | Status | Owner | Target Sprint |
|-----------|--------|-------|---------------|
| Security Audit | 📋 Planned | Security Team | Sprint 21 |
| SOC2 Compliance | 📋 Planned | Compliance + DevOps | Sprint 21-22 |
| Backup & DR Testing | 📋 Planned | DevOps | Sprint 22 |
| Documentation Audit | 📋 Planned | Tech Writers | Sprint 22 |
| User Acceptance Testing | 📋 Planned | QA + Product | Sprint 23 |
| Production Deployment | 📋 Planned | DevOps | Sprint 24 |

**Deliverables**:
- SOC2 compliance report
- Disaster recovery plan tested
- Complete runbooks
- User documentation
- Production deployment successful

---

## Current Sprint (Sprint 12)

### Goals
1. Complete Post Service CRUD operations
2. Implement basic comment functionality
3. Set up notification service foundation
4. Improve test coverage to 70%

### Tasks in Progress

**Post Service** (60% complete):
- ✅ Create post endpoint
- ✅ List posts with pagination
- ✅ Get single post
- 🔄 Update post (owner/admin only)
- 📋 Delete post
- 📋 Image upload to S3

**Comment Service** (Started):
- 🔄 Database schema finalized
- 📋 API design review
- 📋 Implementation

**Notification Service** (20% complete):
- ✅ Email integration (SendGrid)
- 🔄 Event listeners setup
- 📋 Push notification integration
- 📋 Notification preferences

### Team Capacity

| Team | Capacity | Utilization | Notes |
|------|----------|-------------|-------|
| Backend (4 devs) | 160h | 85% | John out sick Mon-Wed |
| Frontend (3 devs) | 120h | 70% | Waiting on API completion |
| Mobile (2 devs) | 80h | 90% | On track |
| QA (2 devs) | 80h | 60% | Need more test scenarios |
| DevOps (1 dev) | 40h | 100% | Infrastructure stable |

---

## Key Performance Indicators (KPIs)

### Development Velocity

| Metric | Target | Current | Trend | Status |
|--------|--------|---------|-------|--------|
| Story Points/Sprint | 60 | 52 | ↓ | ⚠️ Below |
| Sprint Completion | 90% | 85% | → | ⚠️ Below |
| Bug Escape Rate | < 5% | 3% | ↓ | ✅ Good |
| Code Review Time | < 4h | 6h | ↑ | ⚠️ Action |

### Quality Metrics

| Metric | Target | Current | Trend | Status |
|--------|--------|---------|-------|--------|
| Test Coverage | 80% | 65% | ↑ | 🔄 Improving |
| Critical Bugs | 0 | 2 | → | ⚠️ Action |
| P1/P2 Bugs | < 5 | 8 | ↑ | ⚠️ Action |
| Tech Debt Ratio | < 5% | 7% | ↑ | ⚠️ Action |

### Documentation Coverage

| Area | Target | Current | Status |
|------|--------|---------|--------|
| Architecture | 100% | 100% | ✅ Complete |
| API Documentation | 100% | 100% | ✅ Complete |
| Security Docs | 100% | 80% | 🔄 In Progress |
| Feature Specs | 90% | 45% | ⚠️ Behind |
| Runbooks | 100% | 30% | ⚠️ Behind |

---

## Active Blockers

### Critical (Sprint Impact)

1. **Post Service Image Upload Performance**
   - **Issue**: Large images (>5MB) causing timeout
   - **Impact**: Blocks Sprint 12 completion
   - **Owner**: Backend Team
   - **Action**: Investigating CloudFront CDN + S3 direct upload
   - **Target Resolution**: Sprint 12 (This week)
   - **Related**: `/features/blog-posts/`, ADR-007 (pending)

2. **OAuth2 Provider Approval**
   - **Issue**: Google OAuth credentials pending approval
   - **Impact**: Auth testing blocked
   - **Owner**: Security Team
   - **Action**: Following up with Google
   - **Target Resolution**: Sprint 13
   - **Related**: `/features/user-authentication/`, `/docs/security/authentication.md`

### Major (Future Sprint Impact)

3. **Mobile App Store Approval**
   - **Issue**: Need Apple Developer account approval (2-3 weeks)
   - **Impact**: Mobile launch delayed to Sprint 14
   - **Owner**: Mobile Team Lead
   - **Action**: Application submitted
   - **Target Resolution**: Sprint 13
   - **Related**: `/features/mobile-apps/`

4. **Database Migration Strategy**
   - **Issue**: Need to decide on Flyway vs Liquibase
   - **Impact**: Blocks automated migration setup
   - **Owner**: Backend + DevOps
   - **Action**: Tech evaluation meeting scheduled
   - **Target Resolution**: Sprint 12
   - **Related**: `/docs/decisions/adr-006-migration-tool.md` (pending)

---

## Risks & Mitigation

| Risk | Probability | Impact | Mitigation | Owner |
|------|-------------|--------|------------|-------|
| MVP launch delay | Medium | High | Add buffer sprint, reduce scope | PM |
| Key developer departure | Low | High | Cross-training, documentation | EM |
| Security vulnerability | Medium | Critical | Security audit in Sprint 21 | Security |
| Performance issues | High | High | Load testing in Sprint 17 | DevOps |
| Third-party API downtime | Medium | Medium | Retry logic, fallbacks | Backend |

---

## Dependencies & Integrations

### External Services

| Service | Purpose | Status | Risk |
|---------|---------|--------|------|
| SendGrid | Email notifications | ✅ Integrated | Low |
| AWS S3 | Image storage | ✅ Integrated | Low |
| Firebase | Push notifications | 📋 Planned | Medium |
| Google OAuth | Authentication | 🔄 Pending approval | High |
| GitHub OAuth | Authentication | 📋 Planned | Low |
| Stripe | Payments (future) | 📋 Not started | Low |

### Internal Services

| Service | Depends On | Status |
|---------|-----------|--------|
| API Gateway | Auth Service | ✅ Working |
| Post Service | Auth, Database, S3, RabbitMQ | 🔄 In Progress |
| Comment Service | Auth, Post, Database | 📋 Planned |
| Notification Service | Auth, RabbitMQ, SendGrid | 🔄 In Progress |
| Admin Service | Auth, All Services | 📋 Planned |

---

## Budget & Resources

### Development Costs (Estimated)

| Category | Sprint Budget | YTD Actual | Remaining | Notes |
|----------|--------------|------------|-----------|-------|
| Engineering | $80K/sprint | $960K | $320K | 4 sprints remaining (MVP) |
| Infrastructure | $5K/sprint | $60K | $20K | AWS costs increasing with usage |
| External Services | $2K/sprint | $24K | $8K | SendGrid, monitoring tools |
| Contingency | $10K/sprint | $50K | $90K | For unexpected issues |

**Total MVP Budget**: $1.4M  
**Actual Spend**: $1.094M (78%)  
**Projected**: $1.36M (on budget)

---

## Feature Completion Status

Quick reference - see `/features/README.md` for details

| Feature | Status | Progress | Sprint |
|---------|--------|----------|--------|
| User Authentication | ✅ Complete | 100% | Sprint 10 |
| Blog Posts | 🔄 In Progress | 60% | Sprint 12 |
| Comments | 🔄 Started | 10% | Sprint 12-13 |
| Notifications | 🔄 In Progress | 20% | Sprint 13 |
| User Profiles | 📋 Planned | 0% | Sprint 13 |
| Follow System | 📋 Planned | 0% | Sprint 14 |
| Admin Panel | 📋 Planned | 0% | Sprint 15-16 |
| Content Moderation | 📋 Planned | 0% | Sprint 16 |
| Mobile Apps | 🔄 In Progress | 40% | Sprint 12-14 |
| Search | 📋 Planned | 0% | Sprint 19 |

**Legend**:
- ✅ Complete: Feature live in staging
- 🔄 In Progress: Active development
- 📋 Planned: Spec complete, not started

---

## Recent Changes

### Sprint 12 Updates (November 5, 2024)
- Post service image upload moved to Sprint 13 (performance issues)
- Comment service started early (2 days ahead)
- Mobile push notifications delayed to Sprint 14 (approval pending)
- Added ADR-007 decision for CDN strategy (in review)

### Sprint 11 Updates (2025-10-29)
- User authentication completed ✅
- API Gateway rate limiting implemented
- Security documentation 80% complete
- Sprint velocity dropped to 52 (target: 60)

---

## Documentation Links

**For detailed information**:
- **Features**: `/features/README.md` - Individual feature status and tasks
- **Architecture**: `/docs/architecture/README.md` - System design
- **Decisions**: `/docs/decisions/README.md` - ADR log
- **Security**: `/docs/security/README.md` - SOC2 compliance
- **Runbooks**: `/runbooks/README.md` - Operations procedures

**Quick Navigation**:
- New to project? → [Setup Tutorial](tutorials/getting-started/setup-development-environment.md)
- Starting a task? → Find feature in `/features/` folder
- Need API info? → [API Reference](docs/api/rest-api.md)
- Deployment? → [Deployment Runbook](runbooks/deployment.md)

---

## Meeting Schedule

| Meeting | Frequency | Participants | Purpose |
|---------|-----------|--------------|---------|
| Daily Standup | Daily 9:30 AM | Dev Team | Sync, blockers |
| Sprint Planning | Bi-weekly Monday | All | Plan sprint work |
| Sprint Review | Bi-weekly Friday | All + Stakeholders | Demo completed work |
| Sprint Retro | Bi-weekly Friday | Dev Team | Process improvement |
| Architecture Review | Weekly Tuesday | Architects, Leads | Tech decisions |
| Product Sync | Weekly Wednesday | Product, EM | Priority alignment |

---

**Maintained By**: Engineering Manager + Product Manager  
**Updated**: End of each sprint  
**Next Review**: Sprint 13 Planning (November 11, 2024)

---

## How to Use This Roadmap

**Product/Management**: Focus on Phase progress, KPIs, and blockers  
**Developers**: Check current sprint goals and feature links  
**QA**: Review feature completion status and test coverage  
**DevOps**: Monitor infrastructure costs and dependencies  
**Stakeholders**: Review release timeline and risks

**For detailed task tracking**: Navigate to specific feature folder in `/features/`
