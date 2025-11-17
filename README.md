# Diataxis Documentation Framework Template

Welcome to your team's **gold standard** documentation reference repository! This repository provides a complete blueprint for organizing technical documentation using the **Diataxis framework**.

**🎯 New here?** Jump straight to the **[Blog_Engine_Example/](./Blog_Engine_Example/)** - a complete, practical sample showing the Diataxis framework in action for a real-world project.

## 📚 What is Diataxis?

Diataxis is a systematic approach to technical documentation authoring that organizes content into four distinct categories based on user needs and context. Created by Daniele Procida, it helps teams create documentation that is both comprehensive and easy to navigate.

### The Four Quadrants

```
                    Learning-oriented       |    Understanding-oriented
                    (Study)                 |    (Comprehension)
                    
     PRACTICAL      📖 TUTORIALS            |    💡 EXPLANATION
     STEPS          Learning-oriented       |    Understanding-oriented
                    Acquiring skills        |    Theoretical knowledge
                    
                    ─────────────────────────────────────────────────
                    
                    🔧 HOW-TO GUIDES        |    📋 REFERENCE
                    Problem-oriented        |    Information-oriented
                    Achieving goals         |    Describing machinery
                    
     THEORETICAL    Goal-oriented           |    Information-oriented
     KNOWLEDGE      (Work)                  |    (Information)
```

#### 1. 📖 Tutorials (Learning-Oriented)
- **Purpose**: Take users through a series of steps to complete a project
- **Analogy**: Teaching a child to cook
- **Focus**: Learning by doing
- **Example**: "Your First REST API in 30 Minutes"

#### 2. 🔧 How-To Guides (Problem-Oriented)
- **Purpose**: Guide users through solving a specific real-world problem
- **Analogy**: A recipe in a cookbook
- **Focus**: Practical solutions to specific problems
- **Example**: "How to Implement OAuth2 Authentication"

#### 3. 📋 Reference (Information-Oriented)
- **Purpose**: Technical descriptions of the system's machinery and operations
- **Analogy**: An encyclopedia article
- **Focus**: Accurate, complete technical information
- **Example**: "API Endpoint Reference", "Configuration Parameters"

#### 4. 💡 Explanation (Understanding-Oriented)
- **Purpose**: Clarify and illuminate topics, providing background and context
- **Analogy**: An essay on the history of cooking
- **Focus**: Understanding concepts and design decisions
- **Example**: "Understanding Microservices Architecture"

---

## 💡 See It in Action

**Want to see Diataxis in practice?** Check out the **[Blog_Engine_Example/](./Blog_Engine_Example/)** folder - a complete, real-world example of a project documented using the Diataxis framework.

This practical sample demonstrates:
- ✅ How to organize documentation for a realistic multi-platform application
- ✅ All four Diataxis quadrants working together (Tutorials, How-To, Reference, Explanation)
- ✅ Production-ready content (not just placeholders)
- ✅ Best practices for technical writing
- ✅ Documentation for multiple roles (developers, QA, DevOps, management)

**👉 [Explore the Blog Engine Example →](./Blog_Engine_Example/README.md)**

---

## 🗂️ Repository Structure

```
.
├── README.md                          # This file
├── CONTRIBUTING.md                    # Guidelines for documentation authors
├── STYLE_GUIDE.md                     # Writing style and formatting standards
├── FOLDER_STRUCTURE.md                # Detailed structure explanation
├── PROJECT_SUMMARY.md                 # Project overview and goals
├── QUICK_REFERENCE.md                 # Quick reference guide
├── VALIDATION_CHECKLIST.md            # Documentation validation checklist
├── IMPLEMENTATION_COMPLETE.md         # Implementation status and usage guide
├── NEW_PROJECT_START_HERE.md          # Bootstrap guide for brand-new projects
│
├── templates/                         # Blank templates for each doc type
│   ├── tutorial-template.md
│   ├── how-to-template.md
│   ├── reference-template.md
│   └── explanation-template.md
│
├── tutorials/                         # Learning-oriented documentation
│   ├── README.md                      # Overview of available tutorials
│   ├── getting-started/
│   │   ├── 01-setup-development-environment.md
│   │   └── 02-your-first-feature.md
│   └── advanced/
│       └── building-microservices-tutorial.md
│
├── how-to/                           # Problem-oriented guides
│   ├── README.md                     # Index of how-to guides
│   ├── deployment/
│   │   ├── deploy-to-production.md
│   │   └── rollback-deployment.md
│   ├── troubleshooting/
│   │   └── debug-performance-issues.md
│   └── integration/
│       ├── integrate-third-party-api.md
│       └── publish-events.md
│
├── reference/                        # Information-oriented documentation
│   ├── README.md                     # Reference documentation index
│   ├── api/
│   │   ├── rest-endpoints.md
│   │   └── authentication.md
│   ├── configuration/
│   │   ├── environment-variables.md
│   │   └── message-queue.md
│   └── architecture/
│       └── system-components.md
│
└── explanation/                      # Understanding-oriented content
    ├── README.md                     # Explanation documents index
    ├── concepts/
    │   ├── event-driven-architecture.md
    │   └── data-consistency.md
    └── decisions/
        └── why-we-chose-microservices.md
```

### 📦 Example Project

The `/Blog_Engine_Example/` folder contains a complete reference implementation demonstrating Diataxis principles in practice:

- **Complete documentation set** for a realistic multi-platform blog engine
- **All four quadrants** demonstrated with comprehensive examples
- **Multi-role coverage** (BAs, Developers, QA, DevOps, Management)
- **Realistic technical content** (Node.js, PostgreSQL, Kubernetes, microservices)
- **Production-ready protocols** (API conventions, error handling, event schema, logging)
- **Security documentation** (data protection, authorization, API security)
- **Architecture Decision Records** (microservices, PostgreSQL, JWT, RabbitMQ, Redis)
- **Feature-based documentation** (requirements, design, testing)
- **Operational runbooks** (deployment, monitoring, incident response)
- **Use as template** for your own project documentation

**Start here**: 
- `Blog_Engine_Example/README.md` - Overview and structure
- `Blog_Engine_Example/GETTING_STARTED.md` - Quick onboarding guide
- `Blog_Engine_Example/PROJECT_OVERVIEW.md` - Detailed project information

---

## 🚀 Quick Start for Documentation Authors

### Starting a New Project

If you are setting up a **brand-new project** using this repository as a template:

- Begin with **`NEW_PROJECT_START_HERE.md`** for the step-by-step bootstrap guide.
- Ensure each feature has documentation under `features/<feature-id>/` (requirements, design, testing).
- For every feature, **cross-link documentation and Jira**:
  - In docs, include the primary Jira issue/epic (e.g., in a `## Jira` section or in the References block).
  - In Jira, link back to the relevant documentation files (repository URLs).

This keeps written descriptions and tracked work connected, and prevents drift between tickets and documentation.

### Step 1: Identify the Type of Documentation Needed

Ask yourself:
- **Is the user learning?** → Write a **Tutorial**
- **Is the user solving a specific problem?** → Write a **How-To Guide**
- **Is the user looking up technical details?** → Write **Reference** documentation
- **Is the user trying to understand concepts?** → Write an **Explanation**

### Step 2: Choose the Right Template

Navigate to the `/templates/` folder and copy the appropriate template:
- `tutorial-template.md` - For step-by-step learning experiences
- `how-to-template.md` - For problem-solving guides
- `reference-template.md` - For technical specifications
- `explanation-template.md` - For conceptual explanations

### Step 3: Place Your Document in the Correct Folder

Follow the directory structure above. Create subfolders as needed to organize related documents.

### Step 4: Follow the Style Guide

Read `STYLE_GUIDE.md` for formatting, tone, and writing standards.

### Step 5: Review Existing Examples

- Each section contains sample documents demonstrating best practices
- For a complete project example, see `/Blog_Engine_Example/` - a full reference implementation

---

## 📖 Using This Repository

### For Documentation Authors
1. Read `CONTRIBUTING.md` for detailed guidelines
2. Review sample documents in each category to understand structure and tone
3. Use templates as starting points for new documentation
4. Follow the style guide for consistency

### For Documentation Consumers
- **New to the system?** Start with **Tutorials**
- **Need to accomplish a task?** Check **How-To Guides**
- **Looking for technical details?** Consult **Reference** docs
- **Want to understand why/how?** Read **Explanations**

---

## ✅ Quality Checklist

Before submitting documentation, ensure:

- [ ] Document is in the correct Diataxis category
- [ ] Title clearly indicates content and purpose
- [ ] Structure follows the appropriate template
- [ ] Tone matches the document type (imperative for tutorials/how-tos, neutral for reference/explanation)
- [ ] Code examples are tested and working
- [ ] Links to related documentation and relevant Jira tickets are included
- [ ] Document follows style guide conventions
- [ ] Spelling and grammar are correct

---

## 🎯 Key Principles

### Don't Mix Categories
Each document should belong to ONE category. If you find yourself mixing:
- Tutorial steps with reference details → Split them
- How-to guides with explanations → Create separate docs

### User-First Approach
Always consider:
- **Who** is the audience?
- **What** do they need to accomplish?
- **When** will they use this document?
- **Why** do they need this information?

### Maintain Consistency
- Use templates
- Follow naming conventions
- Apply consistent formatting
- Use the same terminology across docs

---

## 📚 Additional Resources

- [Official Diataxis Documentation](https://diataxis.fr/)
- [Diataxis Video Introduction](https://www.youtube.com/watch?v=t4vKPhjcMZg)
- Internal: See `CONTRIBUTING.md` for detailed authoring guidelines
- Internal: See `STYLE_GUIDE.md` for writing standards

---

## 🤝 Contributing

All team members are encouraged to contribute! See `CONTRIBUTING.md` for:
- How to propose new documentation
- Review process
- Maintenance guidelines
- How to report issues with existing docs

---

## 📝 Document Lifecycle

1. **Draft** - Initial creation following templates
2. **Review** - Peer review for accuracy and clarity
3. **Published** - Available to all users
4. **Maintained** - Regular updates as system evolves
5. **Deprecated** - Marked obsolete when no longer relevant

---

## 🏆 Best Practices

### For All Documentation
- Use clear, concise language
- Include code examples where appropriate
- Link to related documents
- Keep documents focused and single-purpose
- Update regularly to match system changes

### Tutorials
- Start with prerequisites clearly stated
- Guide users step-by-step with no assumptions
- Include expected outcomes for each step
- End with a complete working example

### How-To Guides
- State the problem clearly upfront
- Provide the most direct solution
- Include troubleshooting tips
- Link to reference docs for details

### Reference
- Be accurate and complete
- Use consistent formatting
- Include all parameters, return values, errors
- Provide brief code examples

### Explanation
- Provide context and background
- Explain the "why" not just the "what"
- Use diagrams where helpful
- Connect concepts to practical use cases

---

## 📞 Support

Questions about documentation standards? Contact the documentation team or open an issue in this repository.

**Remember**: Good documentation is a team effort. Every contribution makes our system easier to use and understand!

---

*Last Updated: Nov 17, 2025*  
*Framework Version: Diataxis 1.3*  
*Repository Maintainer: Jay A. Leon
