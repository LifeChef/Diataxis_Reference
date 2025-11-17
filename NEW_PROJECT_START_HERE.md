# New Project Start Here (Diataxis + AI-Ready)

**Purpose:** This document is the starting point for **every new project** that uses this repository as its documentation template.

It answers a different question than the `Blog_Engine_Example/` example:

- `Blog_Engine_Example/` → *"How do we document and evolve an existing system?"*
- `NEW_PROJECT_START_HERE.md` → *"We are starting fresh. What do we create first, and in what order, so the team can work effectively (and we can optionally plug in AI later)?"*

---

## 1. Who should read this

- **Tech Leads / Architects** – responsible for initial structure & decisions
- **Product / BA** – shaping the first features and requirements
- **Documentation Owner / Tech Writer / QA2** – owning docs quality

If you are kicking off a *brand new* product, service, or codebase and are adopting this Diataxis structure, **start here before writing any documentation elsewhere.**

---

## 2. Mental Model for New Projects

A new project has three layers of documentation from day one:

1. **Project Core** – identity, scope, and constraints
2. **Standards & Structure** – Diataxis folders, style guide, contributing rules
3. **First Feature Flow** – one concrete feature fully documented end-to-end

You do **not** need all possible docs before coding.
You **do** need a **minimal, stable, non-drifting skeleton** that you grow over time (and can later make AI-ready if desired).

---

## 3. Day 0–1 Checklist (Minimal Viable Documentation)

By the end of your first 1–2 days, aim to have **these artifacts in place**.

> Treat this as your **"repo initialization" sprint for docs**.

### 3.1. Create / Confirm the Base Structure

In your new project repository:

- **Keep / adopt the root meta docs** (adapt titles & content as needed):
  - `README.md` – project-specific intro
  - `CONTRIBUTING.md` – how to add/maintain docs
  - `STYLE_GUIDE.md` – shared writing standards
  - `FOLDER_STRUCTURE.md` – folder map and conventions
  - `QUICK_REFERENCE.md` – how to choose doc type
  - `VALIDATION_CHECKLIST.md` – repo QA

For a detailed list of which files are per-project core vs template-only vs example-only, see `COPY_GUIDE.md`.

- **Keep the Diataxis folders** (initially mostly empty):
  - `/tutorials/`
  - `/how-to/`
  - `/reference/`
  - `/explanation/`

- **Keep the templates**:
  - `/templates/`
    - `tutorial-template.md`
    - `how-to-template.md`
    - `reference-template.md`
    - `explanation-template.md`

> Use `Blog_Engine_Example/` only as **reference**; you dont need a copy of it in each new project.

### 3.2. Define Project Identity (Core Docs)

Create or update these docs **for your actual project**:

- **Root `README.md`**
  - One-paragraph description of the system
  - High-level architecture sketch (even if aspirational)
  - Link to your project summary document (for example, `PROJECT_OVERVIEW.md` or `PROJECT_SUMMARY.md`, if present)

- **Project summary document** (for example, `PROJECT_OVERVIEW.md` or `PROJECT_SUMMARY.md`)
  - What this system is
  - Who it serves
  - Main capabilities (today + near-term)
  - Links to key documentation sections
  - Use this repository's `PROJECT_SUMMARY.md` only as a structural example, not as content to copy verbatim

- **Optional but recommended:**
  - `PROJECT_OVERVIEW.md` or `docs/architecture/overview.md` for richer architecture context
  - `GETTING_STARTED.md` for role-based onboarding (dev, QA, PM, etc.)

### 3.3. Connect to Standards (Avoid Spec Drift)

From day one, clarify where your **canonical standards** live:

- **Security, Privacy, A11y, Observability** **must not** be redefined per project.
- In your new project repo, create **one section** (often in root `README.md` or `docs/`):

```markdown
## Standards & Canonical References

- Security: [link to central security standard]
- OAuth/SSO: [link to central auth standard]
- Data Classification & Privacy: [link]
- Accessibility (A11y): [link]
- Observability & SLO Catalog: [link]
```

- Every feature-level doc should **link to these**, not restate them.

This is what reduces **document drift** and **redundancy**.

### 3.4. Bootstrap the First Feature (End-to-End)

Pick **one flagship feature** for the first iteration (e.g., "user registration", "first order", "tracking a meal").

Create a folder:

```text
features/
  <feature-id>/
    requirements.md
    design.md
    testing.md
```

Use the templates and your team to draft these **based on your actual domain**:

- `requirements.md` (BA / Product)
  - User stories
  - Acceptance criteria
  - Non-functional requirements
  - **References block** (links to standards + existing docs)
  - **Jira link(s)** (primary ticket or epic for this feature)

- `design.md` (Lead Dev / Architect)
  - High-level architecture
  - Component boundaries
  - Data model sketches
  - Integration points

- `testing.md` (QA / Dev)
  - Test strategy for this feature
  - Critical paths
  - Automation plans

> This is your **first complete Diataxis slice**: one concrete feature with clear requirements, design, and testing. Everything else can grow from here.

#### 3.4.1. Link Documentation and Jira Tickets

For every feature:

- Create at least one **Jira issue or epic** that represents the feature.
- In each feature doc (`requirements.md`, `design.md`, `testing.md`), either:
  - Include a **Jira section** (e.g., `## Jira` → `- [PROJ-123](https://jira.example.com/browse/PROJ-123)`), or
  - Include the Jira link inside the **References** block (see below).
- In Jira, add links back to the relevant documentation files (e.g., repository URL) so the relationship is two-way.

This ensures there is always a clear link between written descriptions and tracked work.

### 3.5. Seed Diataxis with One Doc per Quadrant

To make the repo **human-friendly early** (and easy to extend later), create:

- **1 Tutorial** – e.g., `tutorials/getting-started/01-setup-development-environment.md`
- **1 How-To Guide** – e.g., `how-to/deployment/deploy-to-staging.md`
- **1 Reference Doc** – e.g., `reference/architecture/system-components.md` (even if high-level)
- **1 Explanation Doc** – e.g., `explanation/decisions/why-we-chose-architecture-X.md`

Use the templates from `/templates/` and **keep them minimal** at first. You can expand as the system matures.

---

## 4. (Optional) Preparing for Future AI Use

This section is **optional for later**. You can ignore it until you decide to introduce AI assistance.

When you are ready, your goal is not only good docs for humans, but **structured inputs for AI agents**.

### 4.1. Stable Structure & Naming

- Keep folder names and patterns consistent with this template.
- Avoid ad-hoc folders (`misc`, `notes`, `old`, etc.).
- Use kebab-case filenames that reflect purpose, not temporary work.

### 4.2. References Blocks in Feature Docs

In each `features/<feature-id>/*.md`, include:

```markdown
## References
- Standards: [security standard], [auth standard], [A11y standard]
- Architecture: ./docs/architecture/overview.md
- Related Features: ./features/<other-feature>/requirements.md
- API Reference: ./reference/api/rest-endpoints.md
```

This makes it trivial for an AI agent to:

- Read the right context first
- Avoid hallucinating new standards
- Extend existing patterns instead of inventing new ones

### 4.3. Prompt Metadata for AI Tasks

When you use AI to generate or extend docs, include prompt metadata at the top of the file (or in a separate log) similar to:

```yaml
---
model_tier: small|flagship
provider: corporate-ai-platform
model: claude-<ver> | gpt-<ver>
temperature: 0.2
context_id: project-<id>-feature-<id>
mode: create_new|extend|modify
references:
  - ./features/<feature-id>/requirements.md
  - ./docs/architecture/overview.md
---
```

This helps both **auditability** and **future AI runs**, and aligns with the AI workflows shown in `Blog_Engine_Example/WORKFLOW_AGILE_AI.md`.

### 4.4. Assessment-First, Even for New Projects

For any AI-generated artifact, follow this pattern:

1. **Assessment** – AI scans existing docs (even if few) and any external specs.
2. **Generation** – AI drafts the new doc or code, referencing what it read.
3. **Validation** – secondary AI + human review.
4. **Integration** – update cross-links and references.

You can treat this document as the **entry point** for that assessment.

---

### 5. Suggested First-Week Plan for a Brand-New Project

### Day 1–2: Structure & Core Docs

- [ ] Copy / adopt this template into the new project repo
- [ ] Update root `README.md` and `PROJECT_SUMMARY.md`
- [ ] Add "Standards & Canonical References" section
- [ ] Confirm `/templates/` and Diataxis folders exist

### Day 3–4: First Feature Slice

- [ ] Pick 1 flagship feature
- [ ] Create `features/<feature-id>/requirements.md` (BA / Product)
- [ ] Create `features/<feature-id>/design.md` (Lead Dev / Architect)
- [ ] Create `features/<feature-id>/testing.md` (QA / Dev)
- [ ] Ensure each doc has a **References** block **including Jira link(s)**

### Day 5: Seed Diataxis & Workflows

- [ ] Create 1 tutorial, 1 how-to, 1 reference, 1 explanation
- [ ] (Optional, later) Add prompt metadata to docs once AI is introduced
- [ ] Run `VALIDATION_CHECKLIST.md` for a basic pass
- [ ] Capture any gaps or open questions in `PROJECT_STATUS.md` or similar

---

## 6. How This Relates to `Blog_Engine_Example`

Use `Blog_Engine_Example/` as **a worked example**, not a mandatory structure:

- Look at it when you want to see **what "good" looks like** for:
  - Feature docs (`features/comment-system/*`)
  - System-wide docs (`docs/decisions/*`, `docs/security/*`)
  - Runbooks (`runbooks/deployment.md`)
  - AI-assisted workflows (`WORKFLOW_AGILE_AI.md`)

- For a new project:
  - **Do not** blindly copy all files.
  - **Do** copy **patterns**: how docs are named, linked, and structured.

This `NEW_PROJECT_START_HERE.md` file is your **starting map**; `Blog_Engine_Example` is a **full-color atlas** you consult when you need concrete examples.

---

## 7. When Youre "Ready" as a New Project

You can consider your new project **properly bootstrapped** when:

- [ ] Root docs exist and mention Diataxis + standards
- [ ] At least one feature has `requirements`, `design`, and `testing` docs
- [ ] Each feature doc has a **References** section
- [ ] Each Diataxis quadrant has **at least one** project-specific doc
- [ ] AI usage is documented (prompt metadata + workflows)
- [ ] `VALIDATION_CHECKLIST.md` passes all structural items

After that, you can safely lean on `Blog_Engine_Example/WORKFLOW_AGILE_AI.md` and similar documents to run **faster, AI-augmented feature delivery cycles** without losing control of your documentation.

---

*Last Updated: 2025-11*

*Intended Use: Copy or adapt this file into every new project that uses this repository as its documentation foundation.*
