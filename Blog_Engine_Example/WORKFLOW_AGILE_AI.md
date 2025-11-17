# Modern AI-Augmented Feature Delivery (Example run ≈ 2 weeks)

**Last Updated**: Nov 5, 2025  
**Owner**: CTO / Engineering Team  
**Status**: Reference Example (Production-Ready)  

**TL;DR:** This is a concrete example of a feature delivered using AI-assisted development, Diataxis documentation, and reference-first patterns. It serves as a model for how we build, document, test, and ship features—quickly, safely, and with shared knowledge as the byproduct.

> **Worked Example.** This doc shows one successful 2-week run.  
> **For rules & gates:** see *AI-Augmented Delivery – Operating Guide*.  
> **For prompts/CI snippets:** see *Appendix*.  

**Feature Request**: "We need threaded comments with replies and SSO authentication (Google / GitHub)"

**Timeline**: 2 weeks (10 business days)  
**Team**: BA, Lead Dev (FS), FS Developer, FE Developer, Designer, QA1, QA2  
**AI Tools**: Corporate Open WebUI (Claude + ChatGPT), Windsurf (AI-powered IDE for code+docs)  
**Documentation**: Full Diataxis compliance

---

### What This Is
This is an **illustrative example** of how we deliver a feature in 2 weeks using:
- AI-accelerated development
- Diataxis documentation
- Assessment-first and reference-first patterns

It's **not a template** or strict prescription.
Use it as a **pattern** for how AI, humans, docs, testing, and architecture fit together.

**Project-wide Standards:** Security, Privacy, Accessibility (A11y), and Compliance live in the central standards repo.
Do **not** restate them here. Always **link** to the canonical docs to avoid drift.

### How to Use
- Refer to this doc while planning and running your own tasks.
- Don't copy-paste it literally.
- Adapt its order, degree of detail, or stack as needed.
- Keep the same principles: assessment-first, consistent documentation, AI validation, and parallel work.

**Quickstart (medium feature)**
1) Phase-0 (30m): produce `/features/<id>/assessment.md` 
2) Create tickets with **References** + `risk: R1|R2|R3` 
3) Build behind `feature.comments.*` flag (OFF in prod)
4) CI all green (coverage, security, OpenAPI, A11y)
5) Stage + smoke; prepare SLO dashboard links
6) Rollout 5%→25%→50%→100% with rollback criteria

### Standards (Canonical)

| Standard                        | Owner                | Canonical Anchor |
|---------------------------------|----------------------|------------------|
| Security                        | Security Lead        | [TBD: §X]        |
| OAuth/SSO                       | Platform / Auth Lead | [TBD: §Y]        |
| Data Classification & Privacy   | Compliance           | [TBD: §D]        |
| Accessibility (WCAG/A11y)       | Design Systems       | [TBD: §Z]        |
| Observability / SLO Catalog     | SRE                  | [TBD: §O]        |
| Feature Flags / Naming          | Platform Eng         | [TBD: §F]        |

*Standards hosted in central repository (TBD in Standards Repo)*

---

## Core Philosophy

**Always**: Integration-first planning, AI validation, feature flags, documentation standards

**Adapt as needed**: Timeline, specific roles involved, degree of automation, tech stack choices

### Speed Without Sacrificing Quality

1. **AI writes first drafts** → Humans review & refine
2. **Work in parallel** → No sequential handoffs  
3. **Continuous deployment** → Feature flags
4. **Living documentation** → Real-time updates
5. **Cross-validation** → AI checks AI, humans approve
6. **Reference-first** → Link existing specs; don't duplicate
7. **No duplication of standards** → Link to the standard; if the example requires specifics, cite the section (e.g., "OAuth §3.2 Redirect URIs")

### Model Map (used throughout)
| Task type                 | Primary            | Secondary (validation) |
|---------------------------|--------------------|------------------------|
| Extraction / pattern read | GPT-5 (mini)       | —                      |
| Complex reasoning         | GPT-5 (Thinking)   | —                      |
| Codegen                   | SWE-1.5            | Claude 4.5             |
| ADR / docs draft          | ChatGPT            | Claude 4.5             |
| Security review           | Claude 4.5         | Human Lead             |

### Assessment-First Development Policy

**Every task begins by understanding what exists:**

1. **Phase 0: Assessment** (First 30 minutes of ANY task)
   - Windsurf scans existing documentation: `./docs/security/README.md`, `./docs/decisions/*`, `./docs/architecture/*`
   - Windsurf identifies integration points: "Where does this fit in the current system?"
   - ChatGPT analyzes impact: "What existing components are affected?"
   - Human confirms understanding before proceeding

2. **Reference-First Documentation**
   - All new docs reference existing specs (don't duplicate) if a setting exist - we use it from the source, to avoid duplication and spec drift
   - Modifications update existing docs with clear change tracking
   - Every enhancement leaves better documentation for the next task

3. **Windsurf as Documentation Maintainer**
   - Windsurf reads existing docs before creating new ones
   - Windsurf updates docs based on new requirements
   - Windsurf cross-references changes across the documentation set
   - Secondary AI (within Windsurf) validates doc consistency and completeness

### Feature Flagging Standardization

See Feature Flags/Naming Standard for conventions, lifecycle, and propagation (link above).

**Convention**: `feature.<feature-name>.<context>`
- Examples: `feature.comments.global`, `feature.comments.beta-users`, `feature.comments.admin`

**Lifecycle**:
- Default: `OFF` in production until rollout begins
- Development: `ON` for developers in dev environment
- Staging: `ON` for testing in staging environment
- Production: Gradual rollout (5% → 25% → 50% → 100%)

**Environment Propagation**:
- Propagated via `<config-service>` to all services
- Cached at service level with 5-minute TTL
- Fallback to environment variables in local development

**Integration Requirements**:
- All new features must implement feature flag checks
- Feature flags must be logged for audit trails
- Rollback capability must be tested in staging

### Required References Block (in every ticket/spec and feature doc):
```markdown
## References (examples)
- Security: ./docs/security/README.md
- Architecture: ./docs/decisions/adr-001-microservices.md
- Messaging: ./docs/decisions/adr-004-rabbitmq.md
- Component: ./docs/architecture/[relevant-service].md
- Previous Work: [link to related features/tasks]
```

---

## Lean Team (6 People)

| Role | Responsibilities | AI Usage |
|------|------------------|----------|
| **BA** | Requirements, user stories | ChatGPT writes stories → Claude validates |
| **Lead Dev (FS)** | Architecture, ADRs, reviews | Windsurf codes → ChatGPT writes ADRs |
| **FS Developer** | Backend + Frontend | Windsurf development → Claude reviews |
| **FE Developer** | UI/UX implementation, responsive design | Windsurf React → ChatGPT docs |
| **QA Engineer 1** | Automation, security | ChatGPT test cases → Windsurf implements |
| **Designer** | Design systems, Figma handoffs, UX validation | Figma  → Component specs | FE Developer implements |
| **QA Engineer 2** | Manual testing, docs QA, accessibility validation | Claude/ChatGPT troubleshooting |

---

## Reference Pace - Example Flow

> This 2-week flow is a **generic reference**, not a requirement. 
> With AI-accelerated development, many tasks complete **faster**; use judgment based on risk and scope.

This example exists to compare against classic sprint expectations, not to set a deadline.

📌 Anchors  
- Feature flag: `feature.comments.*`  
- Backend: Comment microservice (ADR-006)  
- Frontend: `CommentList`, `CommentForm`, `SSOLoginButtons`  
- Auth: Extend existing OAuth scopes (`comment:write`, `comment:delete`)

**Pacing Guidance**
- Small changes (UI tweaks, minor endpoints): 1–3 days
- Medium features (one service + schema change): ~1 week
- Compound features or regulated changes: up to 2+ weeks
Choose the **lightest** process that still meets standards and SLOs.

### Week 1: Build & Test
- **Day 1-2**: Requirements + Architecture + Setup
- **Day 3-5**: Development (parallel backend/frontend)

### Week 2: Polish & Ship  
- **Day 6-7**: Testing + Bug fixes
- **Day 8-9**: Documentation + Deployment prep
- **Day 10**: Production release

---

## Day-by-Day Workflow - Example Tasks

### At-a-glance flow: BA → (Reqs) → Lead Dev Architecture review/setupDesigner → (Figma/Design System) → FE Dev → (UI Components) → FS Dev → (Backend/API) → Integration → QA → (Tests/Docs) → Lead Dev → (Deploy) → QA2 → (Finalize Docs)

## DAY 1: Requirements & Architecture (Example Tasks)

### Morning (9am-1pm)

#### BA: Requirements (2 hours)
**AI Workflow**: (See Model Map)
0. **Phase 0: Assessment** (Extraction + pattern read + complex reasoning)
1. Feed feature request + assessment findings to ChatGPT (GPT-5 Thinking)
2. ChatGPT generates user stories + acceptance criteria with integration context
3. Claude 4.5 validates (checks for gaps with existing features, edge cases)
4. BA refines & approves

**Inputs:** ./docs/security/README.md • ./features/* • ./docs/decisions/*  
**Outputs:** /features/comment-system/requirements.md (with References)  
**Exit criteria:** Acceptance criteria defined • Risk tag set (R1/R2/R3)

**Prompts**:
```
Windsurf (GPT-5-mini): "Read the existing documentation and identify integration points:
- ./docs/security/README.md (auth patterns)
- ./features/* (existing user workflows)
- ./docs/decisions/* (architectural decisions)
Output: Integration analysis for threaded comments"

ChatGPT (GPT-5 Thinking): "Based on the existing system [assessment findings], generate 5 user stories for threaded comments + SSO.
Include acceptance criteria, success metrics, and integration requirements. +business logic gathered"

Claude 4.5: "Review these requirements against existing system. Flag: conflicts with current auth, missing edge cases, unrealistic metrics, security gaps."
```

**Output**: `/features/comment-system/requirements.md` (includes References section)


#### Lead Dev: Architecture (2 hours)
**AI Workflow**: (See Model Map)
0. **Phase 0: Assessment** (Extraction + pattern read + complex reasoning)
1. Describe system + assessment findings to Windsurf (GPT-5 mini)
2. Windsurf generates technical design that extends existing architecture
3. ChatGPT (GPT-5 Thinking) creates ADR referencing previous architectural decisions
4. Claude 4.5 validates (scalability, security, consistency with existing patterns)
5. Lead Dev approves

**Inputs:** ./docs/decisions/adr-001-microservices.md • ./docs/decisions/adr-004-rabbitmq.md • ./docs/architecture/*  
**Outputs:** /features/comment-system/design.md • /docs/decisions/adr-006-comment-service.md • /reference/database/schema.md (draft)  
**Exit criteria:** Architecture approved • ADR filed • Schema draft ready

**Prompts**:
```
Windsurf (GPT-5-mini): "Read the existing microservice architecture and design comment microservice:
- Follow patterns from ./docs/decisions/adr-001-microservices.md
- Integrate with existing PostgreSQL schemas and Redis patterns
- Use established JWT auth from ./docs/security/README.md
- Connect via RabbitMQ per ./docs/decisions/adr-004-rabbitmq.md
Output: design doc that extends current architecture"

ChatGPT (GPT-5 Thinking): "Based on existing microservice patterns, create ADR for separate Comment Service vs extending Post Service.
Reference: ./docs/decisions/adr-001-microservices.md. Format: standard ADR template."

Claude 4.5: "Review architecture against existing system. Flag: inconsistencies with current patterns, N+1 queries, missing indexes, OAuth vulnerabilities, race conditions."
```

**Output**:
- `/features/comment-system/design.md` (includes References section)
- `/docs/decisions/adr-006-comment-service.md`
- `/reference/database/schema.md` (draft)

### Afternoon (2pm-4pm)

#### Team: Design Review + Task Breakdown (2 hours)
- Review requirements & architecture
- ChatGPT breaks into tasks + estimates
- Windsurf generates design task from requirements
- Create Jira tickets
- Assign work

**AI Workflow**: (See Model Map)
0. **Phase 0: Assessment**
   - **Windsurf (GPT-5-mini)**: "Read existing requirements and architecture documents" (fast extraction)
   - **Windsurf (GPT-5-mini)**: "Analyze current design system patterns and component library" (pattern analysis)
   - **ChatGPT (GPT-5 Thinking)**: "Identify design requirements and constraints from technical specifications" (complex reasoning)
1. **Windsurf (GPT-5-mini)**: Generates comprehensive design task brief (structured generation)
2. Designer reviews and validates design requirements
3. **ChatGPT (GPT-5-mini)**: Creates Jira tickets with clear design acceptance criteria (template-based)

**Design Task Generation**:
```
Windsurf (GPT-5-mini): "Read the approved requirements and architecture documents and generate a comprehensive design task brief:

From ./features/comment-system/requirements.md:
- Extract user stories and acceptance criteria
- Identify UI/UX requirements (threaded layout, forms, interactions)
- Map user flows and interaction patterns
- Note responsive design requirements
- Identify accessibility and performance requirements

From ./docs/decisions/adr-001-comment-service.md:
- Understand technical constraints and capabilities
- Identify component integration points
- Note API endpoints and data structures
- Extract design system extension requirements

Generate design task brief including:
1. Design objectives and success criteria
2. User flows and interaction patterns to design
3. Component requirements (new vs extend existing)
4. Responsive breakpoints and device considerations
5. Accessibility requirements per A11y Standard §Z
6. Integration points with existing design system
7. Deliverables: Figma designs, design tokens, component specs
8. Acceptance criteria for design validation"

Designer: "Review design task brief for completeness and feasibility:
- Validate technical constraints and capabilities
- Identify additional design requirements or considerations
- Confirm deliverables and acceptance criteria
- Estimate design effort and timeline"

ChatGPT (GPT-5-mini): "Create Jira tickets for design work:
- Design task: Comment System UI/UX Design
- Sub-tasks: Figma designs, design tokens, component specs
- Acceptance criteria linked to design brief requirements
- Dependencies on requirements and architecture approval"
```

**Output**: Design task brief in Jira with clear acceptance criteria and deliverables

---

## DAY 2: Design System + Schema + Auth Setup

#### Designer: Design System & Figma Handoff (4 hours)
**AI Workflow**: (See Model Map)
0. **Phase 0: Assessment**
   - Designer: "Review existing design system in Figma and current component library"
   - **Windsurf (GPT-5 (mini))**: "Analyze existing UI patterns and design tokens from current design system"
   - **ChatGPT (GPT-5 (Thinking))**: "Identify how comment components integrate with existing design language"
1. Designer creates/extends Figma designs for comment system
2. Designer generates design tokens and component specifications
3. Windsurf extracts design tokens and generates React component scaffolding
4. Designer validates component scaffolding against Figma designs

**Design Requirements**:
```
Designer: "Create comment system designs in Figma:
- Threaded comment layout (replies, nesting, indentation)
- Comment form (rich text, validation states)
- User avatars and metadata display
- Interaction states (hover, focus, disabled)
- Responsive breakpoints (mobile, tablet, desktop)
- Dark/light theme variations
- Loading and error states

Follow existing design system patterns:
- Use current color palette and typography scale
- Apply existing spacing and border radius systems
- Maintain current button and form field patterns
- Extend existing icon library with comment-specific icons"
```

**Inputs:** Design task brief from DAY 1 • Existing design system • Component library  
**Outputs:** Figma designs • Design tokens • Component specifications  
**Exit criteria:** Designs approved • Design tokens extracted • Component specs ready

---

#### QA Engineer: Design Validation & Test Planning (2 hours)
**AI Workflow**: (See Model Map)
0. **Phase 0: Assessment**
   - **Windsurf (GPT-5 (mini))**: "Review Figma designs against original requirements and technical specifications"
   - **ChatGPT (GPT-5 (Thinking))**: "Identify test scenarios based on design interactions and user flows"
   - **Claude 4.5**: "Validate design completeness and accessibility compliance"
1. QA Engineer validates design-spec alignment
2. QA Engineer creates comprehensive test plan based on designs
3. If issues found → Return to Designer for revisions
4. If approved → Pass to FE Developer for implementation

**Design Validation**:
```
QA Engineer: "Validate Figma designs against requirements:
- Verify all user stories are addressed in designs
- Check interaction flows match technical specifications
- Confirm responsive design covers all required breakpoints
- Validate accessibility compliance (WCAG AA standards)
- Ensure design system consistency maintained

Test Plan Creation:
- User interaction tests (click, type, submit, reply)
- Responsive layout tests (mobile, tablet, desktop)
- Accessibility tests (keyboard navigation, screen readers)
- Visual regression tests (component states, breakpoints)
- Performance tests (animation smoothness, load times)"
```

**Inputs:** Approved Figma designs • Component specifications • Requirements  
**Outputs:** Design validation report • Comprehensive test plan  
**Exit criteria:** Design-spec alignment confirmed • Test plan approved • Ready for FE implementation

---

#### FE Developer: Component Implementation (4 hours)
**AI Workflow**: (See Model Map)
0. **Phase 0: Assessment**
   - **Windsurf (GPT-5 (mini))**: "Extract design tokens and generate React component scaffolding"
   - **ChatGPT (GPT-5 (Thinking))**: "Analyze component integration points with existing frontend architecture"
   - **Claude 4.5**: "Review component API consistency with existing component library"
1. FE Developer implements components following Figma specifications
2. FE Developer integrates components with existing design system
3. FE Developer tests components across all breakpoints
4. QA Engineer validates implementation against test plan

**Component Implementation**:
```
FE Developer: "Implement React components matching Figma specifications:
- CommentThread component (parent container with nested replies)
- CommentItem component (individual comment with metadata)
- CommentForm component (rich text input with validation)
- ReplyButton component (interaction trigger with states)
- UserAvatar component (integration with existing user system)

Implementation requirements:
- Pixel-perfect implementation across all breakpoints
- Accessibility attributes following A11y Standard §Z
- Integration with existing state management patterns
- Performance optimization matching existing components
- Comprehensive unit tests using existing test patterns"
```

**Inputs:** Validated designs • Design tokens • Test plan • Existing component library  
**Outputs:** React components • Unit tests • Component documentation  
**Exit criteria:** Components implemented • Tests passing • QA validation approved

#### Lead Dev: Database (2 hours)
**AI Workflow**: (See Model Map)
0. **Phase 0: Assessment**
   - **Windsurf (GPT-5-mini)**: "Read existing database schemas in ./reference/database/* and migration patterns"
   - **Windsurf (GPT-5-mini)**: "Identify integration points with posts and users tables"
   - **ChatGPT (GPT-5 Thinking)**: "Analyze impact on existing queries and indexes"
1. Windsurf generates migration scripts that extend existing schema
2. Claude reviews (indexes, constraints, foreign key relationships)
3. Test locally → Commit

**Prompt**:
```
Windsurf (GPT-5-mini): "Read existing database schemas and generate PostgreSQL migration for comments:
- Extend current UUID primary key patterns from existing tables
- Add foreign keys to posts.id and users.id following existing FK patterns
- Include audit columns (created_at, updated_at, deleted_at) matching existing tables
- Add indexes on post_id, parent_id following current indexing strategy
- Follow existing migration naming and structure conventions"

Claude 4.5: "Review migration against existing database patterns. Check: foreign key consistency, index naming, constraint patterns, potential performance issues with existing queries."
```

**Output**: `/reference/database/schema.md` (final)

---

#### FS Dev: OAuth Implementation (4 hours)
**AI Workflow**: (See Model Map)
0. **Phase 0: Assessment**
   - **Windsurf (GPT-5-mini)**: "Read existing OAuth implementation in ./docs/security/authentication.md and auth service code"
   - **Windsurf (GPT-5-mini)**: "Identify current OAuth patterns and token management"
   - **ChatGPT (GPT-5 Thinking)**: "Analyze how to extend existing OAuth for new comment permissions"
1. **Windsurf (GPT-5-mini)** extends OAuth2 implementation following existing patterns
2. **ChatGPT (GPT-5 Thinking)** writes security docs that update existing authentication documentation
3. **Claude 4.5** validates OAuth flow consistency with current implementation
4. Test with real providers

**Prompt**:
```
Windsurf (GPT-5-mini): "Read existing OAuth implementation and extend for comment system:
- Follow current OAuth patterns from ./docs/security/authentication.md
- Extend existing /auth/oauth/* endpoints with comment-specific scopes
- Use current JWT token structure and Redis storage patterns
- Maintain existing CSRF protection and state parameter handling
- Add comment permissions to existing token claims structure

Implement OAuth per OAuth Standard §Y (**do not restate details here**).

ChatGPT (GPT-5 Thinking): "Update security docs (scopes/claims) and API reference:
- Extend ./docs/security/authentication.md with new scopes
- Document comment-specific token claims
- Update audience section for comment service
Audience: Developers integrating with existing auth system"

Claude 4.5: "Review OAuth extension against existing implementation. Check: consistency with current patterns, token security, scope minimization, redirect URI validation, integration with existing auth flows."
```

**Output**:
- `/docs/security/authentication.md` (OAuth section)
- `/reference/api/endpoints.md` (auth endpoints)

---

## DAYS 3-4: Core Development (Parallel)

### Backend Team (Lead Dev + FS Dev) - 1.5 days

**AI Workflow**: (See Model Map)
0. **Phase 0: Assessment**
   - Windsurf: "Read existing API patterns in ./reference/api/* and service implementations"
   - Windsurf: "Analyze current Express.js patterns, error handling, and authentication middleware"
   - ChatGPT: "Identify how comment endpoints integrate with existing post and user APIs"
1. Windsurf generates API endpoints + tests extending existing patterns
2. ChatGPT writes API docs that update existing endpoint documentation
3. Claude reviews for vulnerabilities and consistency with current APIs
4. Devs test & refine

**Endpoints** (extending existing API structure):
- `GET /posts/:id/comments` (extends post API pattern)
- `POST /posts/:id/comments` (follows existing creation patterns)
- `POST /comments/:id/reply` (new but follows comment patterns)
- `DELETE /comments/:id` (follows existing soft-delete patterns)

**Prompt**:
```
Windsurf (GPT-5-mini): "Read existing API implementation and extend with comment endpoints:
- Follow Express.js patterns from ./reference/api/endpoints.md
- Use existing authentication middleware from auth service
- Apply current error handling and response formatting
- Extend existing validation patterns for new comment fields
- Implement rate limiting consistent with current APIs

POST /posts/:postId/comments extending post API:
- Auth: JWT required (reuse existing middleware)
- Body: {content, parentId?} (follow existing request patterns)
- Validate: 1-2000 chars (extend current validation approach)
- Return: 201 + comment object (match existing response format)

Include: unit tests following current test structure, Swagger docs updating existing API spec"

Claude 4.5: "Review API extension against existing implementation. Check:
1. Authorization consistency with current endpoint patterns
2. SQL injection protection matching existing safeguards
3. XSS prevention following current sanitization
4. Rate limiting alignment with existing policies
5. N+1 query prevention in threaded comments (analyze existing query patterns)"
```

**Output**: `/reference/api/endpoints.md`

---

### Frontend Team (FE Dev + FS Dev help) - 1 day

**AI Workflow**: (See Model Map)
0. **Phase 0: Assessment**
   - **Windsurf (GPT-5-mini)**: "Read existing component library in ./components/* and design system patterns"
   - **Windsurf (GPT-5-mini)**: "Analyze current React patterns, state management, and styling approach"
   - **ChatGPT (GPT-5 Thinking)**: "Identify how comment components integrate with existing post and user interfaces"
1. **Windsurf (GPT-5-mini)** generates React components + tests extending existing patterns
2. **ChatGPT (GPT-5 Thinking)** writes component docs that update existing component documentation
3. FE Dev integrates with existing design system
4. Manual testing

**Components** (extending existing component library):
- `CommentList` (extends existing list component patterns)
- `CommentForm` (follows existing form patterns)
- `SSOLoginButtons` (extends existing auth button components)

**Prompt**:
```
Windsurf (GPT-5-mini): "Read existing component library and create comment components:
- Follow React patterns from ./components/* (state management, hooks, props)
- Use existing TailwindCSS classes and shadcn/ui patterns
- Apply current form validation patterns to CommentForm
- Extend existing list component patterns for CommentList
- Reuse existing button components for SSO login
- Follow current testing patterns (Jest + React Testing Library)

React TypeScript components for comments:
- CommentList: threaded, pagination, sorting (extend existing list components)
- CommentForm: textarea (2000 max), submit, char counter (follow existing form patterns)
- SSOLoginButtons: Google + GitHub OAuth (extend existing auth button components)

Include unit tests matching current test structure and component documentation"

Claude 4.5: "Review components against existing library. Check:
1. Consistency with current component API and prop patterns
2. Alignment with existing styling and design system
3. Proper integration with existing state management
4. Accessibility following current component standards
5. Performance optimization matching existing patterns"
```

**Output**: `/features/comment-system/design.md` (UI section)

---

## DAY 5: Integration (Example Outputs)

**Team**: All developers  
**Activities**:
- Frontend ↔ Backend integration (extending existing integration patterns)
- Windsurf troubleshoots CORS, auth issues (following existing debugging approaches)
- Set up feature flag (`comments_enabled = OFF`) using existing flag system
- Deploy to staging (following current deployment procedures)
- Smoke tests (extending existing test suites)

**AI Workflow**: (See Model Map)
0. **Phase 0: Assessment**
   - **Windsurf (GPT-5-mini)**: "Read existing integration documentation and staging deployment procedures"
   - **Windsurf (GPT-5-mini)**: "Analyze current CORS configuration and auth integration patterns"
   - **ChatGPT (GPT-5 Thinking)**: "Identify integration points with existing staging environment and monitoring"
1. **Windsurf (GPT-5-mini)** implements integration following established patterns
2. **ChatGPT (GPT-5 Thinking)** updates integration documentation
3. **Claude 4.5** validates integration completeness
4. Team executes integration tests

**Output**: `/features/comment-system/testing.md` (extends existing testing documentation)

---

## DAYS 6-7: Testing & Bug Fixes

### QA1: Automated Testing (1.5 days)

**AI Workflow**: (See Model Map)
0. **Phase 0: Assessment**
   - **Windsurf (GPT-5-mini)**: "Read existing test suites in ./tests/* and current testing patterns"
   - **Windsurf (GPT-5-mini)**: "Analyze current Playwright configurations, Jest setups, and test data management"
   - **ChatGPT (GPT-5 Thinking)**: "Identify how comment tests extend existing user journey and API test patterns"
1. **ChatGPT (GPT-5 Thinking)** generates test cases extending existing test frameworks
2. **Windsurf (GPT-5-mini)** implements automation following current test patterns
3. **Claude 4.5** reviews coverage and consistency with existing test approach
4. Run tests → Log bugs

**Prompt**:
```
Windsurf (GPT-5-mini): "Read existing test infrastructure and implement comment system tests:
- Follow Playwright patterns from ./tests/e2e/* (page objects, fixtures, helpers)
- Use existing Jest configurations and test utilities from ./tests/unit/*
- Apply current test data management and mocking strategies
- Extend existing user journey tests to include comment flows
- Follow current test reporting and CI integration patterns

ChatGPT (GPT-5 Thinking): "Generate test cases for comment system extending existing test coverage:
- Happy paths (create, reply, list) following current user journey patterns
- Edge cases (empty, XSS, SQL injection) extending existing security tests
- Auth tests (unauthenticated, wrong user) building on current auth test suites
- Performance tests (1000 comments load time) using existing performance testing framework

Based on existing test structure in ./tests/*, ensure comprehensive coverage of comment functionality"

Claude 4.5: "Review test implementation against existing test standards. Flag:
- Missing scenarios not covered by current test patterns
- Inconsistencies with existing test data management
- Deviations from current assertion and helper patterns
- Gaps in performance or security testing coverage
- Flaky test patterns following existing test stability guidelines"
```

**Output**: `/features/comment-system/testing.md` (updated)

---

### QA2: Manual Testing + Documentation Review (1 days)

**AI Workflow**: (See Model Map)
0. **Phase 0: Assessment**
   - Windsurf: "Read existing documentation structure and current review processes"
   - Windsurf: "Analyze existing troubleshooting guides and known issues documentation"
   - ChatGPT: "Identify documentation gaps and integration points with existing user guides"
1. Manual exploratory testing following existing test protocols
2. Claude reviews all documentation for consistency with existing docs:
   - Hallucinations (non-existent features)
   - Inconsistencies with current documentation patterns
   - Missing integration points with existing guides
3. ChatGPT generates troubleshooting guide extending existing issue resolution patterns
4. QA2 refines based on actual bugs found and existing documentation standards

**Prompt**:
```
Windsurf: "Read existing documentation structure and prepare for review:
- Analyze current doc patterns in ./docs/*, ./features/*, ./reference/*
- Review existing troubleshooting guides in ./how-to/troubleshooting/*
- Identify documentation standards and formatting conventions
- Check integration points with existing user and developer documentation

Claude: "Review all comment system docs against existing documentation standards:
- /features/comment-system/* (check consistency with existing feature docs)
- /reference/api/endpoints.md (verify integration with current API documentation)
- /docs/security/authentication.md (ensure alignment with existing security docs)
Check: hallucinations, logic gaps, contradictions with existing system documentation"

ChatGPT: "Create troubleshooting guide for comment issues extending existing patterns:
- Follow structure from ./how-to/troubleshooting/* existing guides
- Include integration with existing authentication and post-related issues
- Reference existing solutions and debugging approaches
- Based on QA findings: [paste bug list]
- Ensure consistency with current troubleshooting documentation format"
```

**Output**: `/how-to/troubleshooting/comment-issues.md` (draft)

---

### Developers: Bug Fixes (1 day)
- Triage bugs (12 found: 2 critical, 6 medium, 4 low)
- Fix critical/medium bugs
- Re-test

---

## DAYS 8-9: Documentation Sprint

🧱 **Outcome**: All features should still produce the 4 Diataxis doc types.
🎨 **But how** you split, timebox, assign, or stage the work can vary.

### QA2 (acts as Tech Writer): User Documentation (2 days)

**AI Workflow**: (See Model Map) Generate all 4 Diataxis categories extending existing documentation

0. **Phase 0: Assessment**
   - Windsurf: "Read existing user documentation structure and current Diataxis patterns"
   - Windsurf: "Analyze existing tutorials, how-to guides, and explanation documents"
   - ChatGPT: "Ensure comment documentation integrates seamlessly with existing user learning paths"

#### Tutorial (Learning-Oriented)
**Prompt**:
```
Windsurf: "Read existing tutorial patterns and prepare for comment tutorial:
- Analyze structure from ./tutorials/getting-started/* existing tutorials
- Follow current step-by-step format and screenshot conventions
- Identify integration points with existing user onboarding flow
- Ensure consistency with current tutorial writing style and complexity

ChatGPT: "Write tutorial: 'Your First Comment on a Blog Post' extending existing tutorial patterns:
Target: New users (following current user persona definitions)
Include:
1. Sign in with Google (building on existing auth tutorials)
2. Find a blog post (extending current navigation guides)
3. Leave a comment (new content following existing tutorial structure)
4. Reply to someone else (extending existing interaction patterns)
Format: Match existing beginner-friendly approach, screenshot style, expected results format

Reference: ./tutorials/getting-started/01-setup-development-environment.md for tone and structure"

Claude: "Validate tutorial against existing documentation standards. Check:
- Missing prerequisites compared to current tutorial patterns
- Unclear steps deviating from existing tutorial clarity
- Wrong screenshots or formatting inconsistencies
- Missing troubleshooting integration with existing help patterns
- Alignment with current learning progression and complexity"
```

**Output**: `/tutorials/getting-started/03-your-first-comment.md`

---

#### How-To Guide (Problem-Oriented)
**Prompt**:
```
Windsurf: "Read existing how-to guide patterns and prepare for comment interaction guide:
- Analyze structure from ./how-to/* existing guides
- Follow current problem-statement and solution format
- Identify integration points with existing user interaction documentation
- Ensure consistency with current how-to writing style and complexity level

ChatGPT: "Write how-to: 'Reply to a Comment' extending existing how-to patterns:
Target: Users who know basics (following current user progression)
Format:
- Problem statement (matching existing guide structure)
- Prerequisites (building on existing how-to prerequisites)
- Quick steps (numbered, following current step format)
- Troubleshooting section (integrating with existing help patterns)

Reference: ./how-to/* for format, tone, and complexity alignment"

Claude: "Review how-to guide against existing documentation standards. Check:
- Assumes too much knowledge compared to current how-to patterns
- Missing edge cases covered by existing guides
- Unclear instructions deviating from current how-to clarity
- Integration gaps with existing user interaction documentation
- Consistency with current problem-solving approach"
```

**Output**: `/how-to/interaction/reply-to-comments.md`

---

#### Reference (Information-Oriented)
**Already done during development** (extending existing reference docs):
- `/reference/api/endpoints.md` (API specs - updated with comment endpoints)
- `/reference/database/schema.md` (DB schema - extended with comment tables)

**QA2**: Reviews for accuracy with existing reference docs, updates with Claude's help

**AI Workflow**: (See Model Map)
0. **Phase 0: Assessment**
   - Windsurf: "Read existing reference documentation structure and current formatting patterns"
   - Windsurf: "Analyze current API spec formats and database documentation conventions"
   - ChatGPT: "Ensure comment reference docs integrate seamlessly with existing technical documentation"

#### Explanation (Understanding-Oriented)
**Prompt**:
```
Windsurf: "Read existing explanation documents and prepare for threaded comments explanation:
- Analyze structure from ./explanation/* existing explanations
- Follow current conceptual explanation format and technical depth
- Identify integration points with existing architecture and design explanations
- Ensure consistency with current explanation writing style and audience targeting

ChatGPT: "Write explanation: 'Understanding Threaded Comments' extending existing explanation patterns:
Include:
- What are threaded comments? (building on existing interaction concept explanations)
- Why use threading vs flat? (referencing existing design decision explanations)
- How it works (parent_id relationships - following current technical explanation depth)
- Design decisions (single-level threading - aligning with existing architectural explanation style)
Target: Developers and curious users (matching current explanation audience levels)

Reference: ./explanation/* for tone, technical depth, and format consistency"

Claude: "Validate explanation against existing documentation standards. Check:
- Technical inaccuracies compared to current system explanations
- Missing context from existing architectural and design explanations
- Unclear diagrams or visualizations deviating from current explanation patterns
- Integration gaps with existing conceptual documentation
- Alignment with current technical depth and audience complexity"
```

**Output**: `/explanation/concepts/threaded-comments.md`

---

### Lead Dev: Deployment Runbook (4 hours)

**AI Workflow**: (See Model Map)
0. **Phase 0: Assessment**
   - Windsurf: "Read existing deployment runbooks and current deployment procedures"
   - Windsurf: "Analyze current CI/CD patterns, staging environments, and monitoring setup"
   - ChatGPT: "Identify how comment service deployment integrates with existing deployment workflows"
1. ChatGPT generates deployment checklist extending existing runbook patterns
2. Claude validates for missing steps and consistency with current deployment procedures
3. Lead Dev adds specifics (IP addresses, credentials) following existing runbook format

**Prompt**:
```
Windsurf: "Read existing deployment documentation and prepare for comment service runbook:
- Analyze structure from ./runbooks/* existing deployment guides
- Follow current deployment patterns and procedures
- Identify integration points with existing service deployment workflows
- Ensure consistency with current monitoring and rollback procedures

ChatGPT: "Create deployment runbook for Comment Service extending existing deployment patterns:
- Pre-deployment checklist (building on existing checklists)
- Step-by-step deployment (following current blue-green deployment approach)
- Rollback procedure (integrating with existing rollback patterns)
- Monitoring setup (extending current monitoring and alerting)
- Success criteria (aligning with existing deployment success metrics)

Reference: ./runbooks/* for format, procedures, and operational consistency"

Claude: "Review runbook against existing deployment standards. Flag:
- Missing health checks compared to current service deployment patterns
- No database migration rollback procedures (inconsistent with existing database deployment practices)
- Unclear rollback triggers (deviating from current deployment decision criteria)
- Integration gaps with existing monitoring and alerting systems
- Deviations from current deployment communication and coordination procedures"
```

**Output**: `/runbooks/deployment.md` (updated)

---

## DAY 10: Production Release

### Morning (9am-12pm): Deployment

**Team**: Lead Dev (executes), All (monitor)

**AI Workflow**: (See Model Map)
0. **Phase 0: Assessment**
   - Windsurf: "Read existing production deployment procedures and current monitoring setup"
   - Windsurf: "Analyze current feature flag management and production rollout patterns"
   - ChatGPT: "Identify integration points with existing production deployment coordination"

**Process** (following existing production deployment patterns):
1. Apply database migrations (using existing migration procedures)
2. Deploy Comment Service (containers) following current service deployment workflow
3. Verify health checks (extending existing monitoring setup)
4. Enable feature flag for 5% users (using existing feature flag system)
5. Monitor (errors, latency, user feedback) following current production monitoring practices

**AI Usage**:
- Windsurf monitors logs in real-time using existing log analysis tools
- ChatGPT analyzes error patterns following current error analysis procedures
- Alert if error rate > 1% (integrating with existing alerting thresholds)

### Afternoon (1pm-5pm): Gradual Rollout

**Timeline** (following existing rollout patterns):
- 1pm: 5% → 25% (if no issues)
- 3pm: 25% → 50%
- 5pm: 50% → 100%

**Rollback Trigger** (aligned with existing production criteria):
- Error rate > 5% (following current rollback thresholds)
- P95 latency > 500ms (consistent with existing performance criteria)
- Critical bugs reported (integrating with existing incident response)

**AI Usage**:
- Claude analyzes user feedback (support tickets) using existing feedback analysis tools
- ChatGPT generates FAQ from common questions extending existing FAQ patterns

### Evening: Documentation Finalization

**QA2**: Final doc review
- Update troubleshooting with real user issues (following existing issue documentation patterns)
- Add FAQ section (integrating with existing FAQ structure)
- Final Claude validation (using existing documentation quality standards)

**Output**: All docs finalized and published following existing documentation release procedures

---

## RACI & Approvals (Lean)

- **BA**: Stories (A/R)
- **Lead Dev**: ADRs, architecture, security signoff (A/R)
- **QA2**: Documentation owner (A/R)
- **Developers**: Implementation and tests (R)
- **CTO**: Final approval for ADRs and production release (A)
- **Security/Compliance Lead**: Required approver when changes touch auth, PII/PHI, or regulated workflows.

## Observability & Release Gates

**Conformance Checks (link to standards)**
- Security Review: per Security Standard §X (threat model, authZ, secrets, scans)
- OAuth/SSO: per OAuth Standard §Y (flows, redirect URIs, token lifetimes)
- A11y: per A11y Standard §Z (roles, focus, keyboard nav, color/contrast)
- Observability: per SLO Catalog (endpoint SLIs/SLOs, error budget policy)

**Observability**
- Adopt endpoint SLIs/SLOs from the Observability Catalog.
- If no entry exists, propose SLOs via the catalog process and add dashboard links.

**Release gates**: Staged rollout (5% → 25% → 50% → 100%) proceeds only if SLIs meet SLOs; otherwise trigger rollback  
**Success criteria:** error rate <1% and p95 <500ms during 5%→25%→50% rollout windows; otherwise rollback

## Lean CI/CD Pipeline & Automated Security Gates

**Pipeline stages**: lint → unit/integration → e2e (critical paths) → SAST → dependency & license scan → secrets scan → IaC scan (if applicable) → A11y scan: per A11y Standard §Z (tooling + thresholds defined there) → build → preview → staging

Conformance: Run the checks defined in **Observability & Release Gates → Conformance Checks**.

**Gates (fail build if)**:
- Coverage below threshold (e.g., unit ≥ 70%, integration ≥ 50%)
- Any critical/high vulnerability
- Any exposed secret detected
- OpenAPI/Swagger validation fails or docs link check fails

**Release**: All gates green + CTO approval. No manual security steps outside CI.

## Definition of Ready (DoR)

- **Assessment Complete**: Phase 0 assessment completed with integration analysis
- **References Listed**: All existing docs referenced and linked (security, ADRs, architecture, previous work)
- **Integration Points Identified**: Clear mapping to existing components and systems
- **Acceptance Criteria**: Include NFRs and integration requirements with existing systems
- **Prompt Metadata**: Committed with model tier, provider, temperature, and reference context
- **Feature Flag**: Key defined and rollout plan sketched following existing flag patterns
- **Impact Analysis**: Documented effects on existing services and documentation

## Definition of Done (DoD)

- **Tests**: Meet thresholds extending existing test coverage; critical-path e2e added
- **Security**: Gates green; OAuth checklist passed; conform to Security Standard §X
- **Documentation**: All Diataxis artifacts updated; status set to "approved" following existing doc standards
- **Observability**: Dashboards and alerts from SLO Catalog; runbook linked
- **Integration**: Seamless integration with existing services and documentation confirmed
- **Rollout**: Staged rollout executed or rollback validated following existing procedures
- **Knowledge Transfer**: Existing documentation enhanced for future modification tasks

## Documentation Map (Same as Traditional)

**Visual Legend:**
📘 docs/ → System-Wide  
🛠 features/ → Team-Specific  
🎓 tutorials/ → Learning  
🧩 how-to/ → Problem-Solving  
📚 reference/ → Information  
🧠 explanation/ → Understanding  
🚀 runbooks/ → Operations  

```
Blog_Engine_Example/
├── features/comment-system/          [Team-Specific]
│   ├── requirements.md               (BA + ChatGPT)
│   ├── design.md                     (Lead Dev + Windsurf)
│   └── testing.md                    (QA1 + ChatGPT)
│
├── docs/                             [System-Wide]
│   ├── decisions/
│   │   └── adr-006-comment-service.md  (Lead Dev + ChatGPT)
│   ├── security/
│   │   └── authentication.md           (FS Dev + ChatGPT)
│   └── architecture/services/
│       └── README.md                   (Lead Dev + Windsurf)
│
├── tutorials/                        [Learning]
│   └── getting-started/
│       └── 03-your-first-comment.md    (QA2 + ChatGPT)
│
├── how-to/                           [Problem-Solving]
│   ├── interaction/
│   │   └── reply-to-comments.md        (QA2 + ChatGPT)
│   └── troubleshooting/
│       └── comment-issues.md           (QA2 + ChatGPT)
│
├── reference/                        [Information]
│   ├── api/
│   │   └── endpoints.md                (FS Dev + Windsurf)
│   └── database/
│       └── schema.md                   (Lead Dev + Windsurf)
│
├── explanation/                      [Understanding]
│   └── concepts/
│       └── threaded-comments.md        (QA2 + ChatGPT)
│
└── runbooks/                         [Operations]
    └── deployment.md                   (Lead Dev + ChatGPT)
```

---

## 🤖 AI Tool Usage by Phase

| Phase | Primary AI | Secondary AI | Human Role |
|-------|-----------|--------------|------------|
| **Requirements** | ChatGPT (generates) | Claude (validates) | BA (refines) |
| **Architecture** | Windsurf (designs) | ChatGPT (ADR) | Lead Dev (approves) |
| **Backend Dev** | Windsurf (codes) | Claude (reviews) | Devs (test) |
| **Frontend Dev** | Windsurf (components) | ChatGPT (docs) | FE Dev (integrates) |
| **Testing** | ChatGPT (test cases) | Windsurf (implements) | QA (validates) |
| **Documentation** | ChatGPT (writes) | Claude (validates) | QA2 (refines) |
| **Deployment** | ChatGPT (runbook) | Claude (reviews) | Lead Dev (executes) |

---

## Key AI Workflows

### 1. Code Generation (Windsurf)
```
Developer: "Implement X feature"
   ↓
Phase 0: Assessment - Windsurf reads existing docs and patterns
   ↓
Windsurf: Generates code + tests extending existing architecture
   ↓
Claude: Reviews for bugs/security and consistency with current patterns
   ↓
Developer: Tests, refines, commits
```

### 2. Documentation Generation (ChatGPT)
```
Human: "Document X feature"
   ↓
Phase 0: Assessment - Windsurf analyzes existing doc structure
   ↓
ChatGPT: Generates draft extending current documentation patterns
   ↓
Claude: Validates (hallucinations, gaps, consistency with existing docs)
   ↓
Human: Reviews, edits, approves
```

### 3. Cross-Validation (Claude)
```
Primary AI: Generates artifact based on existing system analysis
   ↓
Claude: Reviews with checklist:
  • Hallucinations?
  • Logic gaps?
  • Security issues?
  • Inconsistencies with existing patterns?
  • Integration completeness?
   ↓
Human: Makes final decision
```

### 4. Assessment-First Development (New Pattern)
```
Task Received
   ↓
Phase 0: Assessment (30 mins)
   - Windsurf: "Read existing docs and identify integration points"
   - ChatGPT: "Analyze impact on current system"
   - Human: "Confirm understanding"
   ↓
Implementation extending existing architecture
   ↓
Documentation enhancing current knowledge base
   ↓
Validation against existing standards
```

---

## AI Prompts Library

 Note: Include prompt front matter in committed files:
 ```yaml
 model_tier: small|flagship
 provider: open-webui
 model: claude-<ver> | gpt-<ver>
 temperature: <0-1>
 context_id: <component_or_adr_number>
 mode: modify|extend|create_new
 references: [list of referenced docs]
 ```

### Requirements
```markdown
Windsurf: "Read existing requirements in ./features/* and security patterns in ./docs/security/*"

ChatGPT: "Generate user stories for [feature] extending existing requirements patterns.
Include: acceptance criteria, integration requirements, impact analysis."

Claude: "Validate requirements against existing system. Flag: conflicts with current patterns, missing integration points, unrealistic metrics."
```

### Architecture
```markdown
Windsurf: "Read existing architecture in ./docs/decisions/* and ./docs/architecture/*"

ChatGPT: "Create ADR for [decision] referencing existing architectural decisions.
Format: Context, Decision, Consequences, Integration Impact."

Claude: "Review architecture for: consistency with existing patterns, scalability, security, integration completeness."
```

### Code
```markdown
Windsurf: "Read existing code patterns and implement [feature] extending current architecture.
Include: tests following existing patterns, error handling matching current approach, docs updating existing specs."

Claude: "Review code for: bugs, security, performance, consistency with existing patterns, integration completeness."
```

### Documentation
```markdown
Windsurf: "Read existing documentation structure and current patterns"

ChatGPT: "Write [doc type] for [feature] extending existing documentation.
Audience: [who]. Format: [existing template]. Integration: [related docs]."

Claude: "Validate docs for: hallucinations, missing integration points, inconsistencies with existing documentation."
```

### Testing
```markdown
Windsurf: "Read existing test infrastructure and patterns"

ChatGPT: "Generate test cases for [feature] extending existing test coverage.
Include: integration tests, security tests following current patterns."

Windsurf: "Implement tests using existing frameworks and patterns."

Claude: "Review test coverage against existing standards. Flag integration gaps and consistency issues."
```

 ---
 
## Risks & Escalation Paths

| Risk | Detection | Escalation Path | Human-Only Mode Trigger |
|------|-----------|-----------------|------------------------|
| **Windsurf misinterprets codebase** | Generated code doesn't compile or breaks existing patterns | 1. Claude review flags inconsistencies<br>2. Human Lead Dev validates integration<br>3. Re-run Phase 0 with more specific context | 3+ failed attempts to integrate with existing patterns |
| **Secondary AI contradicts primary** | Conflicting recommendations in validation | 1. Human reviews both AI outputs<br>2. CTO makes final decision on architectural direction<br>3. Document decision in ADR | Critical architectural disagreements affecting system stability |
| **Bugs in AI-written tests cause flakiness** | Test failures in CI, inconsistent results | 1. QA1 investigates test patterns<br>2. Human writes stable test skeleton<br>3. AI fills in test cases following human framework | >20% test failure rate or flaky tests blocking deployment |
| **AI hallucinations in documentation** | Claude validation flags non-existent features | 1. QA2 manually verifies against actual system<br>2. Update references and context<br>3. Human rewrites problematic sections | Documentation errors causing user confusion or support tickets |
| **Performance degradation in AI-generated code** | Monitoring alerts, slow response times | 1. Lead Dev profiles and identifies bottlenecks<br>2. Human optimizes critical paths<br>3. AI generates optimized versions following human patterns | P95 latency > 2x SLO targets |

### Human-Only Mode Criteria
Switch to human-only development when:
- **Security critical**: Authentication, payments, PII handling
- **Production incidents**: Active outage or critical bug fixes
- **AI failure pattern**: Same issue occurs 3+ times with AI assistance
- **Complex integration**: Requires deep understanding of legacy systems
- **Compliance requirements**: Regulatory or audit scenarios

---

## Getting Started

### Prerequisites
1. **AI Tool Access**:
   - Corporate Open WebUI (Claude + ChatGPT) for chat AI
   - Windsurf (developers)

2. **Team Training** (1 week before sprint):
   - Effective prompt engineering
   - AI validation techniques
   - Tool-specific training

3. **Templates**:
   - Standardized prompts for each doc type
   - Validation checklists for Claude
   - Diataxis templates

### Corporate AI Platform & Model Tiering

- All chat AI usage goes through Corporate Open WebUI exposing Claude and ChatGPT models.
- **Small models** (GPT mini/nano; Claude lightweight): brainstorming, transformations, test case drafts, scaffolding, link checks.
- **Flagship models** (Claude flagship / GPT flagship): architecture reviews, security reviews, production runbooks, high-risk code, data migrations, public docs.
- **Default rule**: start on Small; upgrade to Flagship when tasks are security/PII/auth/crypto/migrations/public docs or the small model shows uncertainty.
- **Metadata**: commit prompt + context + model metadata as front matter (see AI Prompts Library note).
- **Privacy**: do not send PII or raw production logs to AI; use redaction per `./docs/security/README.md`.

### First Sprint Setup
1. Create prompt library (this doc) with assessment-first patterns
2. Set up AI accounts and access to existing documentation repository
3. Train team on cross-validation and integration analysis
4. Run practice feature (1-day exercise) on existing codebase
5. Refine process based on integration learnings

### Modification vs New Feature Decision Tree
```
Task Received
   ↓
Phase 0: Assessment
   ↓
Does this modify existing components?
   ├─ Yes → Follow Modification Path:
   │   - Read existing component docs
   │   - Identify extension points
   │   - Update existing docs (don't create new)
   │   - Focus on integration impact
   └─ No → Follow New Feature Path:
       - Create new feature docs
       - Reference existing patterns
       - Establish new integration points
       - Document new architecture decisions
```

### Example: Modification Path in Action

**Task**: "Add comment deletion API endpoint"

#### Phase 0: Assessment (15 minutes)
```
Windsurf: "Read existing comment API patterns in ./reference/api/endpoints.md"
→ Finds: POST /posts/:id/comments, GET /posts/:id/comments
→ Identifies: Missing DELETE endpoint pattern
→ Analyzes: Existing soft-delete patterns in User Service

ChatGPT: "Analyze impact on existing comment system"
→ Identifies: Need to update comment threading logic
→ Determines: Integration with existing permission system
→ Documents: Effects on comment count calculations
```

#### Implementation (2 hours)
```
Windsurf: "Extend existing comment API with DELETE endpoint:
- Follow current Express.js route patterns from existing endpoints
- Use existing authentication middleware
- Apply current soft-delete approach from User Service
- Update existing comment list queries to handle deleted_at

Prompt includes:
context_id: comment-service-001
mode: extend
references: [./reference/api/endpoints.md, ./docs/security/authentication.md]
```

#### Documentation Updates (30 minutes)
```
Windsurf: "Update ./reference/api/endpoints.md (don't create new file):
- Add DELETE /comments/:id to existing endpoint table
- Update existing authentication notes for comment permissions
- Extend existing error response examples

ChatGPT: "Update ./features/comment-system/design.md:
- Add deletion section to existing UI patterns
- Update existing user flow diagrams
- Extend current permission matrix"
```

#### Key Differences from New Feature Path:
- **No new files created** - only updates to existing docs
- **Integration focus** - how deletion affects existing comment threading
- **Pattern consistency** - follows established soft-delete conventions
- **Reference continuity** - builds on existing API documentation structure

---

## Related Documentation

- [Traditional Workflow](./WORKFLOW_EXAMPLE.md) - 6-week process
- [Contributing Guide](./CONTRIBUTING.md) - Standards
- [Diataxis Templates](../templates/) - Doc templates

## Docs-as-Code Enforcement

- Each doc has an owner and status (draft/reviewed/approved) with links to source tickets.
- Link checker and docs linter run in CI; builds fail on broken links.
- Reference lints: tickets/specs must include a "References" block; CI fails if missing.

**Drift Guardrails**
- CI lints fail if local docs restate Security/A11y requirements instead of linking.
- PR template requires "Standards links" checklist with section IDs (e.g., OAuth §3.2).
- Link checker enforces reachable anchors to canonical sections.

**Evidence Artifacts**
- Attach links to CI runs, security scan reports, and rollout dashboards in the PR description.
- Tag the ADR number and standards sections used (e.g., ADR-006, OAuth §3.2).

---

## FAQ

**Q: Does AI replace technical writers?**  
A: No. QA2 acts as tech writer, using AI to accelerate drafting. Human expertise in clarity, audience, and polish is irreplaceable.

**Q: How do you prevent AI hallucinations?**  
A: Two-step validation: Secondary AI + human review. Phase 0 assessment ensures AI reads existing docs before generating, reducing hallucination risk.

**Q: Can a 6-person team really do this in 2 weeks?**  
A: Yes, with:
- Assessment-first approach (30 mins upfront saves hours downstream)
- Parallel work (no sequential handoffs)
- AI acceleration (3-5x faster drafting)
- Multi-skilled team (less coordination overhead)
- Feature flags (continuous deployment)
- Integration with existing patterns (reduces decision-making)

**Q: What if AI-generated code has bugs?**  
A: Same as human code: testing catches them. AI doesn't reduce testing need, but assessment-first approach ensures consistency with existing patterns.

**Q: Is documentation quality lower?**  
A: No. Same Diataxis standards. AI drafts, humans refine. Often higher consistency due to pattern-based approach.

**Q: How does this work for modifications vs new features?**  
A: Phase 0 assessment determines the path. Modifications update existing docs, new features create new docs but reference existing patterns. Both follow integration-first thinking.

---

## Other Ways to Run This

Variants we've used:
- 1-week spike version (no prod push, just code + doc scaffolding)
- 3-week "compound feature" with 2 parallel workstreams
- 4-day hotfix using only 2 Diataxis doc types (quick ref + how-to)

---

> **Reference vs. Example:** This document shows **how** work can flow.
> For requirements, consult the **standards** above; for pacing, use the **pacing guidance** here.

---

*This workflow is battle-tested in startup environments. Adapt for your context.*
