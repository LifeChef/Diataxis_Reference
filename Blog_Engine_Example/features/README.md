# Features

> **Purpose**: Feature-based task organization and tracking  
> **Audience**: Developers, Product Managers, QA  
> **Last Updated**: November 5, 2024

> **ℹ️ Note**: Items labeled (SAMPLE) are illustrative and may not have corresponding folders yet.

---

## Overview

This folder contains all features organized by functional area. Each feature has its own subfolder with complete context: requirements, design, tasks, testing, and references.

**Why feature folders?**
- ✅ All context in one place
- ✅ Easy for AI assistants to load complete feature context
- ✅ Clear ownership and status tracking
- ✅ Parallel development without conflicts

---

## Feature Status

| Feature | Status | Progress | Sprint | Owner | Priority |
|---------|--------|----------|--------|-------|----------|
| User Authentication (SAMPLE) | ✅ Complete | 100% | Sprint 10 | Backend Team A | P0 |
| [Blog Posts](blog-posts/) | 🔄 In Progress | 60% | Sprint 12 | Backend Team B | P0 |
| Comment System (SAMPLE) | 🔄 Started | 15% | Sprint 12-13 | Backend Team C | P1 |
| Notifications (SAMPLE) | 🔄 In Progress | 25% | Sprint 13 | Backend Team D | P1 |
| User Profiles (SAMPLE) | 📋 Planned | 0% | Sprint 13 | Backend Team A | P1 |
| Admin Panel (SAMPLE) | 📋 Planned | 0% | Sprint 15-16 | Frontend Team | P2 |

**Legend**:
- ✅ Complete: Feature deployed to staging
- 🔄 In Progress: Active development
- 📋 Planned: Spec complete, waiting to start
- ⏸️ Paused: Blocked or deprioritized

---

## Feature Folder Structure

Each feature folder contains:

```
feature-name/
├── README.md         # Overview, status, quick start
├── requirements.md   # User stories, acceptance criteria
├── design.md         # Technical design, architecture
├── tasks.md          # Implementation checklist
├── testing.md        # Test scenarios, edge cases
└── references.md     # Links to ADRs, components, security
```

**README.md** - Feature overview
- What is this feature?
- Current status and progress
- Owner and sprint info
- Quick links to other files

**requirements.md** - What to build
- User stories ("As a..., I want..., so that...")
- Acceptance criteria
- Out of scope items
- Dependencies on other features

**design.md** - How it works
- Technical architecture
- Data models
- API contracts
- Sequence diagrams
- Technology choices

**tasks.md** - Implementation checklist
- Granular tasks (1-4 hours each)
- Task status (todo, in progress, done)
- Owner assigned to each task
- References to components/ADRs

**testing.md** - Test strategy
- Unit test scenarios
- Integration test scenarios
- Edge cases and error conditions
- Performance requirements
- Security test cases

**references.md** - Related documentation
- Architecture Decision Records (ADRs)
- Components/services used
- Security requirements
- API endpoints
- Database tables

---

## How to Use

### Starting a New Feature

1. **Create folder structure**:
   ```bash
   mkdir features/new-feature
   cd features/new-feature
   touch README.md requirements.md design.md tasks.md testing.md references.md
   ```

2. **Fill in requirements.md first**:
   - Define user stories
   - Get product approval

3. **Design.md next**:
   - Technical design
   - Architecture review

4. **Then tasks.md**:
   - Break design into tasks
   - Estimate each task

5. **Update this README**:
   - Add feature to table above
   - Set status to "Planned"

### Working on a Feature

1. **Start sprint**:
   - Update feature status to "In Progress"
   - Assign tasks to developers

2. **During development**:
   - Mark tasks as done in `tasks.md`
   - Update progress % in feature `README.md`
   - Reference feature folder in PRs

3. **Ready for testing**:
   - QA follows `testing.md` test scenarios
   - Fix bugs, update docs

4. **Feature complete**:
   - Mark status as "Complete"
   - Update main roadmap
   - Celebrate! 🎉

### For AI Assistants

**To implement a feature**, load entire feature context:
```bash
cat features/blog-posts/*.md
cat docs/decisions/adr-001-microservices.md   # From references.md
cat docs/security/authorization.md            # From references.md
```

**This gives complete context**:
- What to build (requirements)
- How to build it (design)
- What tasks remain (tasks)
- What to test (testing)
- What patterns to follow (references → ADRs)

---

## Quick Links

**Roadmap**: [/ROADMAP.md](../ROADMAP.md) - High-level project status  
**Architecture**: [/docs/architecture/](../docs/architecture/) - System design  
**Decisions**: [/docs/decisions/](../docs/decisions/) - ADRs  
**Security**: [/docs/security/](../docs/security/) - Security controls

---

**Maintained By**: Product Manager + Engineering Manager  
**Review Frequency**: Weekly during sprint planning  
**Last Review**: November 5, 2024
